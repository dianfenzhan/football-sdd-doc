# 功能清单（以代码为准）

个人主体小程序，不含 AI / 魔镜。小程序端不直连库，一律走云函数 `api`。

## 页面（`miniprogram/app.json`）

| 页面 | 主要调用 |
| --- | --- |
| home | `me`、`homeSummary`（下一场固定 `clubId=all`：仅当前用户已报名正选的最近一场 + 天气；无则各角色统一「暂无报名的比赛」）；首页无新建入口（新建只在活动页 FAB）；界龙展示报名进度条；已报名横幅界龙分队后显示本场队名（未分队仅「已报名」，待分队显示「待分队」）；天气先 Open-Meteo（首请求+1 次重试），再云函数 `activityWeather`，仍失败显示「点此重试」 |
| activities | `me`、`listActivities`、`deleteActivity`；顶栏标题「活动」字号固定；多队时绿顶栏下小字当前队 + ▾，贴顶下拉切球队；`startTime + 12h` 已过的场次列表置灰仍可点进详情 |
| activity | `activityDetail`、`signup`、`cancelSignup`、`setWaitUntil`、`saveProfile`、`updateQuota`、`saveRound`、`deleteRound`、`setContribution`、`deleteContribution`、`setAbsent`、`saveTacticsBoard`；未加入周三联或无分组时点报名弹注册/入队窗（带出已有头像、昵称、擅长位置）；报名须知后申请订阅消息，失败/取消/超时仍继续报名；分队场管理员可见「点人数上限改名额」提示 |
| activity-edit | `me` / `orgMe`、`createActivity` |
| stats | `me`、`homeSummary`、`clubStats`；顶栏「数据」字号固定；多队时绿顶栏下小字当前队 + ▾ 切球队；含进球、助攻、乌龙、出勤；鸽子榜仅管理员可见，并提示「仅管理员可见，此信息不会暴露给队员」 |
| players | `me`、`listPlayers`、`adminUpdatePlayer`、`setClubAdmin`；顶栏「球员」字号固定；多队时绿顶栏下小字当前队 + ▾ 切球队；「全部」只读并提示先切球队；列表不展示进球等数据，管理员点队员进弹窗才看/改本队运动数据与资料/设管理员；普通成员不返回也不展示 `clubStats` |
| me | `me`、`orgMe`、`saveProfile`、`setMyGroup`、`updateClubGroup`；含个人运动数据栏（总活动数、总进球数、总助攻数、总乌龙球数、总鸽子数） |
| club | `joinClub`、`updateClubGroup`、`reorderClubGroups`；顶栏「球队」字号固定；多队时绿顶栏下小字当前队 + ▾ 切球队 |
| session | `activityDetail`、`saveProfile`、`signup`、`cancelSignup`、`updateActivityBasics`、`adminMoveSignup` / `adminSwapSignup`、`addSessionTeam` / `removeSessionTeam`、`setSessionColor`、`saveTacticsBoard`、`startJielongCup`、`clearFixtureScores`、`saveFixtureScore`、`addFixture`、`deleteRound`、`deleteRoundDirectory`、`resetFixtures`、`updateFixture`、`setContribution`、`setAbsent`、`deleteContribution`；仅未加入界龙时点报名弹注册/入队窗（带出已有头像、昵称、擅长位置）；已入队（含名册为空按「未分组」）不再反复确认资料；入队后走报名须知（鸽子规则，界龙开赛前 1.5h）并申请开赛提醒订阅；报名不设人数上限、无后补；新建场默认带 A/B 空队且 `manualRoster=true`，报名一律进「未分队池」（`team_pending`）；仅管理员可拉人进组或调队；界面无「按组分队」；「开启界龙杯」放在分队网格下方，不满足 4/5 非空队且 ≥24 人时置灰仍可点出原因；队名默认 A～F 队，改名可选名册组自动带出球衣色（可再改）；4 队小组赛积分榜 + 决赛/季军赛记满后显示领奖台，5 队小组赛全部记满后按积分显示冠亚季；其余战术板/记分/重开界龙杯行为同前 |

## 云函数 action

### 身份与成员

| action | 要点 |
| --- | --- |
| me | 当前用户资料、权限与个人运动数据汇总（`myStats`：总活动数、总进球数、总助攻数、总乌龙球数、总鸽子数） |
| saveProfile | 个人资料（姓名最多 16 字、头像、擅长位置选填）与球队分组解耦；支持传 `clubId + groupId`（或 `teamId`）按球队入队并落组，不污染未加入球队；非法值失败 |
| listMembers | 仅可授权管理员 |
| listPlayers | 球员列表；调用者为某队管理员/主管理员时，对可管球队附带 `clubStats`（该队总活动数、总进球数、总助攻数、总乌龙球数、总鸽子数）；普通成员无此字段 |
| setAdmin | 设管理员 |
| adminUpdatePlayer | 管理员改球员资料；改名同样最多 16 字 |

### 活动与报名

| action | 要点 |
| --- | --- |
| listActivities | 分页活动列表，可按俱乐部过滤 |
| activityDetail | 详情、报名、比分、战术板等 |
| createActivity | 标题/地点必填；分队或联合 |
| deleteActivity | 仅授权人（主管理员）可删；界龙亦要求主管理员；普通管理员无删除入口 |
| updateQuota | 分队改名额；满员后补可顶上；联合场拒绝 |
| signup / cancelSignup | 报名与取消；临期取消记鸽子（周三联开赛前 4h、界龙 1.5h；开赛前重报清除）；界龙报名不设上限、无后补，须已 joinClub；管理员介入分队/调人后新报名进 `team_pending` 未分队池 |
| moveSignup / swapSignups | 仅联合场 |
| setWaitUntil | 后补等待时长 |
| smokeSeedSignups | 冒烟灌数，非业务入口 |
| smokeSetCallerRole | 仅测试云：切换当前用户为 owner/admin/member/fresh，供三角色走查；正式环境拒绝 |

### 战绩与战术

| action | 要点 |
| --- | --- |
| saveRound / deleteRound | 轮次比分新增/修改与删除（`deleteRound` 兼容周三联与界龙场次删除） |
| setContribution / deleteContribution | 整场进球、助攻、乌龙球记录与删除（两队通用；系统鸽子与管理员鸽子均可删除） |
| setAbsent | 鸽子；系统鸽子不可直接改状态（可删除）；有进球/助攻/乌龙时不可标鸽子 |
| saveTacticsBoard | 通用活动分组战术板（周三联与界龙共用），按 `activityId + teamId` 持久化（改组名不影响已存板），仅允许排入本组正选，默认 8 人 3-2-2、场上最多 9 槽；未报名不可存 |
| stats | 周期战绩表 |
| homeSummary | 下一场 = 调用者已报名正选且未开赛的最近一场（首页 `clubId=all`；可按俱乐部筛）；无报名则 `nextActivity` 为空；附带近期榜（进球/助攻/乌龙/出勤/鸽子），榜单可按俱乐部/队过滤；鸽子榜仅对应队管理员（或主管理员）返回，非管理员 `absencesTop` 为空，并带 `absencesAdminOnly: true` |
| activityWeather | 按开赛时间取天气 |

### 多俱乐部 / 场次组织

| action | 要点 |
| --- | --- |
| orgMe / joinClub / setMyGroup / switchClub | 俱乐部身份 |
| updateClubGroup / reorderClubGroups / setClubAdmin | 组与管理员 |
| updateActivityBasics | 界龙管理员开赛前改标题/地点/时间 |
| assignTeams | 界龙按组一键分 6 队并开启手动分队池模式（`manualRoster=true`），不自动生成对阵；有比分后锁队 |
| adminMoveSignup / adminSwapSignup | 界龙管理员调人（仅管理员可点选或拖拽队员换组/对调、从「未分队」池拉人进组，操作后置 `manualRoster=true`）；对调互换队伍与顺序；换组写入组内末位 `sessionOrder` |
| addSessionTeam / removeSessionTeam | 分队后加队（最多 6）/ 删空队（至少留 2），操作后置 `manualRoster=true`；删队时若该队仍在对阵中则拒绝；对阵展示找不到队名时兜底为默认色名并标注「已不在名单」，不再直接露出 `sess_*` id |
| startJielongCup / clearFixtureScores | 按键开启/重开界龙杯（校验非空正式队为 4 或 5 支且队内正选 ≥24 人，置 `cupStarted=true`、`cupTeamIds`、`manualRoster=true` 并生成轮次对阵；已开赛后若不满足 4/5 队或 24 人可传 `exitToManual=true` 清除轮次与积分榜退回手动模式；有比分时拒绝开赛/重开）；`clearFixtureScores` 一键清空本场全部已记比分（保留轮次与个人数据） |
| saveFixtureScore / updateFixture / addFixture / deleteFixture / deleteRoundDirectory / resetFixtures / setSessionColor | 场次比分（`clear` 或两侧空分可清空；开启界龙杯且 4 队前 3 轮记满后自动补第 4 轮冠亚/季军赛）、新增/修改/删除场次、删整轮并前移后续 `roundNo`、未记分时重排/清空对阵、对调双方、换场地、改队名/配色 |
| addGoalEvent / deleteGoalEvent | 历史进球事件接口（兼容保留） |
| clubStats | 俱乐部统计（汇总 `contributions` 与历史 `goal_events` 的进球、助攻、乌龙） |

### 其它

| action | 要点 |
| --- | --- |
| sendKickoffReminders | 定时开赛提醒（周三联提前 2h，界龙提前 2h） |
| devEcho | 仅确认打到本地代码 |

已移除：`mirrorAsk`、`mirrorFeedback`、`mirrorQuota` 及前端悬浮助手。
