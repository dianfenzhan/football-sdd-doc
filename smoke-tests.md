# 冒烟检查（以 `codeBase/scripts/smoke-test.js` 为准）

前提：开发者工具自动化端口、当前 profile 云环境可用。不含 AI / 魔镜。

## 不测

- 真实订阅消息送达
- 线上部署与密钥轮换
- 云库集合手工建表以外的控制台操作

## 断言表（检查名）

身份与资料：`me`、`me myStats`、`saveProfile`、`saveProfile empty name`、`saveProfile name too long`、`saveProfile bad team`、`saveProfile bad position`、`saveProfile bad jielong group`、`saveProfile personal only`、`listMembers` / `listMembers denied`、`listPlayers`、`listPlayers sorted desc`、`listPlayers admin clubStats`、`adminUpdatePlayer`、`adminUpdatePlayer position`

分队报名与后补：`createActivity split`、`smokeSeedSignups signed`、`signup full without choice`、`signup choice other when full`、`cancelSignup choice other`、`signup split waitlist`、`setWaitUntil`、`setWaitUntil bad hours`、`updateQuota promote waitlist`、`updateQuota promotes waitlist`、`updateQuota below signed count`、`moveSignup on split`、`swapSignups on split`、`cancel promotes waitlist`

联合报名：`createActivity joint`、`activityDetail joint`、`updateQuota joint denied`、`signup joint own group`、`moveSignup`、`moveSignup to unite`、`swapSignups`、`swapSignups exchanged`、`signup joint home full to unite`、`signup joint shared waitlist`

鸽子与取消：`cancel waitlist no pigeon`、`early cancel no pigeon`、`late cancel pigeon hidden until kickoff`、`setAbsent system pigeon locked`、`re-signup cleared system pigeon`

战绩与首页：`saveRound` / `saveRound updated` / `round removed`、`setContribution` / `contribution removed`、`setAbsent`、`homeSummary` 系列（已报名下一场 / after cancel no next / span / team / weather；shape 含 `absencesAdminOnly`）、`activityWeather missing time`、`stats:` 增量

战术板：`saveTacticsBoard too many`、`saveTacticsBoard activity`、`activity tactics board saved`、`saveTacticsBoard unsigned`

活动生命周期：`createActivity`（含空标题/地点失败）、`signup` / `signup duplicate`、`listActivities`、`activity listed`、`activityDetail after delete`

界龙：`orgMe two clubs`、`createActivity jielong`、`createActivity jielong pending shell`、`signup jielong pending`、`smokeSeedSignups jielong`、`pending pool before assign`、`assignTeams by roster six`、`smokeSeedSignups jielong after manual`、`activityDetail divided`、`addFixture manual`、`deleteFixture manual`、`deleteRoundDirectory renumber`、`addSessionTeam full`、`jielong tactics board kept after rename`、`removeSessionTeam occupied`、`removeSessionTeam empty`、`startJielongCup too few active teams`、`startJielongCup four active teams`、`saveFixtureScore cup round1`、`startJielongCup locked by score`、`clearFixtureScores`、`resetFixtures four teams`、`startJielongCup and clearFixtureScores`

已移除：`mirrorAsk` / `mirrorFeedback` / `mirrorQuota` 相关检查。

## 三角色 UX 走查（`npm run role:ux`）

仅测试云。用 `smokeSetCallerRole` 切换身份后断言权限与页面 chrome；覆盖普通管理、普通成员、仅周三联进界龙入队窗、未加入任何队、恢复授权人。上线把关清单见 `doc/qa-prelaunch.md`。
