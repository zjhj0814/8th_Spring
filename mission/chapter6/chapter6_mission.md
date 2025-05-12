## 미션
# 1. N+1 문제를 해결할 수 있는 여러 가지 다른 방법들에 대해 조사한 후, [ 핵심 키워드 ] 에 정리
→ 정리 완료.
# 2. 2주차 미션 때 했던 해당 화면들에 대해 작성했던 쿼리를 QueryDSL로 작성하여 리팩토링하기
## 1. 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)
사용자미션 생성시간 DESC, 미션 id DESC
```sql
SELECT m.id, m.missionSpec, m.reward, mm.status, s.name
FROM mission AS m
JOIN member_mission AS mm ON m.id = mm.mission_id
JOIN store AS s ON m.store_id = s.id
WHERE mm.member_id = 'member_id' 
  AND mm.status = 'member_mission_status'
  AND (
    mm.created_at < 'cursor_created_at'
    OR (mm.created_at = 'cursor_created_at' AND m.id < 'cursor_mission_id')
  )
ORDER BY mm.created_at DESC, m.id DESC
LIMIT 15;
```
```java
BooleanBuilder predicate = new BooleanBuilder();

predicate.and(memberMission.memberId.eq('memberId'));
predicate.and(memberMission.status.eq('status'));

if ('cursorCreatedAt' != null && 'cursorMissionId' != null) {
    predicate.and(
        memberMission.createdAt.lt('cursorCreatedAt')
        .or(memberMission.createdAt.eq('cursorCreatedAt')
            .and(mission.id.lt('cursorMissionId')))
    );
}

return queryFactory
        .select(Projections.constructor(MyMissionDto.class,
                mission.id,
                mission.missionSpec,
                mission.reward,
                memberMission.status,
                store.name
                ))
        .from(mission)
        .join(memberMission)
        .join(store)
        .where(predicate)
        .orderBy(memberMission.createdAt.desc(), mission.id.desc())
        .limit(15)
        .fetch();
```
## 2. 리뷰 작성하는 쿼리(사진의 경우 일단 배제)
```sql
INSERT INTO review (store_id, member_id, score, body, reply, created_at, updated_at) 
VALUES ({store_id}, {member_id}, {score}, {body}, null, {created_at}, {updated_at});
```
리뷰를 insert하는 쿼리의 경우, ReviewRepository(Interface)를 사용해도 충분할 것 같다.
```java
Review review = Review.builder()
        .store('store')
        .member('member')
        .score('score')
        .body('body')
        .build();
reviewRepository.save(review);
```
## 3. 홈 화면 쿼리(현재 선택된 지역에서 도전이 가능한 미션 목록, 페이징 포함)
미션 데드라인 ASC, 미션 id ASC
```sql
SELECT m.id, m.store_id, m.missionSpec, m.reward, m.deadline, s.name
FROM store AS s
JOIN mission AS m ON s.id = m.store_id
WHERE s.region_id = 'region_id'
  AND m.deadline > NOW()
  AND m.id NOT IN (
    SELECT mission_id
    FROM member_mission
    WHERE member_id = 'member_id'
  )
  AND (
    m.deadline > '{cursor_deadline}'
    OR (m.deadline = '{cursor_deadline}' AND m.id > 'cursor_mission_id')
  )
ORDER BY m.deadline ASC, m.id ASC
LIMIT 5;
```

```java
BooleanBuilder predicate = new BooleanBuilder();

predicate.and(store.regionId.eq('regionId'));
predicate.and(mission.deadline.gt(LocalDateTime.now()));

//JPAExpressions는 다양한 위치에 들어갈 수 있는 서브쿼리를 만들기 위해 사용된다.
predicate.and(
    mission.id.notIn(
        JPAExpressions
            .select(memberMission.mission.id)
            .from(memberMission)
            .where(memberMission.member.id.eq('memberId'))
    )
);

if ('cursorDeadline' != null && 'cursorMissionId' != null) {
    predicate.and(
        mission.deadline.gt('cursorDeadline')
            .or(mission.deadline.eq('cursorDeadline')
                .and(mission.id.gt('cursorMissionId')))
    );
}

return queryFactory
        .select(Projections.constructor(HomeMissionDto.class,
            mission.id,
            store.id,
            mission.missionSpec,
            mission.reward,
            mission.deadline,
            store.name
            ))
        .from(mission)
        .join(store)
        .where(predicate)
        .orderBy(mission.deadline.asc(), mission.id.asc())
        .limit(5)
        .fetch();
}
```
## 4. 마이 페이지 화면 쿼리
```sql
SELECT m.id, m.nickname, m.email, m.point,
    CASE 
           WHEN m.is_auth = 'Y' THEN m.phone_num
            ELSE '미인증'
    END AS phone_num
FROM member AS m WHERE id = {member_id};
```
```java
return queryFactory
        .select(Projections.constructor(MyPageDto.class,
                member.id, 
                member.nickname, 
                member.email, 
                member.point,
                new CaseBuilder()
                        .when(member.is_auth.eq("Y")).then(member.phoneNum)
                        .otherwise("미인증")
                        .as("phone_num")
        )
        .from(member)
        .where(member.id.eq('memberId'))
        .fetchOne();
```