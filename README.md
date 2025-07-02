#요구사항

1. 하루 동안 얼마나 많은 사용자들이 <로그인 화면>에 접속하는가

2. <이벤트 광고> 화면을 가장 많이 본 사용자는 누구인가

3. <메인 화면>을 가장 많이 보는 시간대는 하루 중에 언제인가

4. 메뉴 1-2-3 화면 중에서 가장 전환을 많이하는 화면은 어디서/어디로 전환하는 경우인가.

5. 지난 일주일 동안 메뉴 2 마지막 화면에서 값을 저장하고 <메인 화면>으로 이동한 횟수는 몇 번인가

6. 하루 동안 메뉴 3 마지막 화면에서 ON/OFF 설정을 선택한 사용자는 몇 명인가

7. 최근 일주일 기간에 가장 화면 노출이 적은 화면은 어느 화면인가

#기록을 저장하기위한 장치를 먼저 만들기
<무엇이 발생했는가?>
1. "로그인 화면에 들어왔다" → 접속 기록
2. "가장 많이 본 사용자" -> 접속기록
3. "가장 많이 본 시간대" -> 접속기록
4. "화면에서 화면으로 이동했다" → 화면 전환 기록
5. "메인화면으로 이동한 횟수" -> 화면 전환 기록
---->gpt제안 : "저장"하고 이동했기때문에 저장(행동기록)과 전환기록을 함께 수행했을 때를 기록
6. " on/off를 눌렀다." -> 행동기록
7. " 노출이 가장 적은 화면" -> 접속기록

#스스로 세워보는 체크 포인트
- 문제해결에 필요한 데이터 구조 설계에대해 학습했다.
- 문제해결을 위한 적절한 데이터 구조와 프로그래밍 흐름을 설계할 수 있다.
- 데이터 구조에 필요한 데이터 타입을 적절하게 선택할 수 있다.


class UserTracker {
  constructor() {
    this.logs = [];
  }

// 1. 화면 접속 기록
  logView(userId, loginscreen) {
    this.logs.push({
      type: "view",//키는 타입, 밸류는 뷰
      userId : userId,//키 : 유저아이디, 밸류 : 유저아이디
      screen: loginscreen,//키 : 스크린, 밸류 : 로그인스크린
      time: new Date()//키:타임:,밸류 :시간전체
    });
  }

  // 2. 화면 전환 기록 (from → to)
  logTransition(userId, fromScreen, toScreen) {
    this.logs.push({
      type: "transition",
      userId,
      from: fromScreen,
      to: toScreen,
      time: new Date()
    });
  }

  // 3. 행동 기록 (ex: 값 저장, ON/OFF 선택 등)
  logAction(userId, screen, action) {
    this.logs.push({
      type: "action",
      userId,
      screen,
      action,
      time: new Date()
    });
  }

  // 4. 통계용 도우미: 하루 기준으로 날짜 문자열 구하기
  //정확한 날짜는 필요없지만 하루 단위로 비교하거나 묶기위해서 필요
  _getDayStr(date) {
    return date.toISOString().slice(0, 10); // YYYY-MM-DD
  }

  //1. 하루 동안 로그인 화면 접속한 사용자 수
  countLoginUsersForToday() {
    const today = this._getDayStr(new Date());
    const users = new Set();

    for (let log of this.logs) {
      if (
        log.type === "view" &&
        log.screen === "login" &&
        this._getDayStr(log.time) === today
      ) {
        users.add(log.userId);
      }
    }

    return users.size;
  }

  // 2. 이벤트 광고를 가장 많이 본 사용자
  mostFrequentUserForEventAd() {
    const count = {};
    for (let log of this.logs) {
      if (log.type === "view" && log.screen === "event-ad") {
        count[log.userId] = (count[log.userId] || 0) + 1;
      }
    }

    let maxUser = null;
    let maxCount = 0;
    for (let user in count) {
      if (count[user] > maxCount) {
        maxCount = count[user];
        maxUser = user;
      }
    }

    return maxUser;
  }

  // 3. 메인 화면 가장 많이 보는 시간대
  mostFrequentHourForMain() {
    const hourCount = Array(24).fill(0);

    for (let log of this.logs) {
      if (log.type === "view" && log.screen === "main") {
        const hour = new Date(log.time).getHours();
        hourCount[hour]++;
      }
    }

    let maxHour = 0;
    for (let i = 1; i < 24; i++) {
      if (hourCount[i] > hourCount[maxHour]) {
        maxHour = i;
      }
    }

    return maxHour;
  }

  // 4. 메뉴 1/2/3 중 전환이 가장 많은 from → to
  mostFrequentTransitionInMenus() {
    const transitions = {};

    for (let log of this.logs) {
      if (log.type === "transition") {
        const { from, to } = log;
        const isMenu = screen =>
          ["menu-1", "menu-2", "menu-3"].includes(screen);
        if (isMenu(from) && isMenu(to)) {
          const key = `${from}→${to}`;
          transitions[key] = (transitions[key] || 0) + 1;
        }
      }
    }

    let maxKey = null;
    let maxVal = 0;
    for (let k in transitions) {
      if (transitions[k] > maxVal) {
        maxVal = transitions[k];
        maxKey = k;
      }
    }

    return maxKey;
  }

  // 5. 지난 7일 동안 메뉴2 마지막 화면에서 값 저장 후 메인으로 이동한 횟수
  countSaveThenMainLast7Days() {
    const now = Date.now();
    let count = 0;

    for (let i = 0; i < this.logs.length - 1; i++) {
      const log1 = this.logs[i];
      const log2 = this.logs[i + 1];

      const within7Days = (log) =>
        now - new Date(log.time).getTime() < 7 * 24 * 60 * 60 * 1000;

      if (
        log1.type === "action" &&
        log1.screen === "menu-2-last" &&
        log1.action === "save" &&
        log2.type === "transition" &&
        log2.from === "menu-2-last" &&
        log2.to === "main" &&
        within7Days(log1) &&
        within7Days(log2)
      ) {
        count++;
      }
    }

    return count;
  }

  // 6. 오늘 하루 동안 menu-3-last에서 ON/OFF 한 사용자 수
  countToggleUsersToday() {
    const today = this._getDayStr(new Date());
    const users = new Set();

    for (let log of this.logs) {
      if (
        log.type === "action" &&
        log.screen === "menu-3-last" &&
        (log.action === "ON" || log.action === "OFF") &&
        this._getDayStr(log.time) === today
      ) {
        users.add(log.userId);
      }
    }

    return users.size;
  }

  // 7. 최근 7일간 가장 노출이 적은 화면
  leastViewedScreenLast7Days() {
    const now = Date.now();
    const viewCount = {};

    for (let log of this.logs) {
      if (
        log.type === "view" &&
        now - new Date(log.time).getTime() <= 7 * 24 * 60 * 60 * 1000
      ) {
        viewCount[log.screen] = (viewCount[log.screen] || 0) + 1;
      }
    }

    let minScreen = null;
    let minCount = Infinity;

    for (let screen in viewCount) {
      if (viewCount[screen] < minCount) {
        minCount = viewCount[screen];
        minScreen = screen;
      }
    }

    return minScreen;
  }
}
