# 삼성 SW 역량 테스트 기출 문제 (백준)
## 13460. 구슬 탈출 2
> https://www.acmicpc.net/problem/13460

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <cmath>
using namespace std;

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

struct Pos {
  int y, x;
  bool operator==(const Pos& rhs) const {
    return y == rhs.y && x == rhs.x;
  }
  int dist(const Pos& rhs) const {
    return abs(y - rhs.y) + abs(x - rhs.x);
  }
};

struct Info {
  Pos red, blue;
  bool move(int dir);
};

int N, M;
char board[10][10];
Pos hole;
void init(Info& info);

int answer = 11;
void dfs(Info& info, int cnt);

int main() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);
  Info info;
  init(info);
  dfs(info, 0);
  cout << (answer == 11 ? -1 : answer) << '\n';
  return 0;
}

void dfs(Info& info, int cnt) {
  if (cnt >= answer) return;
  if (info.red == hole) {
    answer = cnt;
    return;
  }
  for (int dir = 0; dir < 4; ++dir) {
    Info moved = info;
    if (moved.move(dir)) {
      dfs(moved, cnt + 1);
    }
  }
}

bool Info::move(int dir) {
  Pos n_red = red, n_blue = blue;

  // move red
  while (board[n_red.y + dy[dir]][n_red.x + dx[dir]] != '#') {
    n_red.y += dy[dir];
    n_red.x += dx[dir];
    if (n_red == hole) break;
  }

  // move blue
  while (board[n_blue.y + dy[dir]][n_blue.x + dx[dir]] != '#') {
    n_blue.y += dy[dir];
    n_blue.x += dx[dir];
    if (n_blue == hole) return false;
  }

  if (n_red == n_blue) {
    if (red.dist(n_red) > blue.dist(n_blue)) {
      n_red.y -= dy[dir];
      n_red.x -= dx[dir];
    }
    else {
      n_blue.y -= dy[dir];
      n_blue.x -= dx[dir];
    }
  }

  if (red == n_red && blue == n_blue) {
    return false;
  }

  blue = n_blue;
  red = n_red;
  return true;
}

void init(Info& info) {
  cin >> N >> M;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < M; ++j) {
      cin >> board[i][j];
      if (board[i][j] == 'R') {
        info.red = { i, j };
        board[i][j] = '.';
      }
      else if (board[i][j] == 'B') {
        info.blue = { i, j };
        board[i][j] = '.';
      }
      else if (board[i][j] == 'O') {
        hole = { i, j };
        board[i][j] = '.';
      }
    }
  }
}
```
</details>

### 설명
1. 전역변수로 상태를 저장하는 것보다 상태를 복사하고, 복사한 상태에 필요한 작업을 수행한 뒤 참조 형태로 재귀호출하는 것이 더 직관적이다.

2. 재귀호출을 하며 상태를 저장할 때 스택 오버플로우를 고려하여 상태를 힙에 저장하는 방법을 종종 사용했었는데 그럴 필요 없다. dfs의 동작 원리를 생각해보면 상태는 dfs의 최대 깊이 만큼만 생기고, 이 문제에서는 최대 깊이가 10이므로 스택에는 Info 구조체가 최대 10개만 쌓인다.

3. 보드를 같은 방향으로 연속해서 두 번 이상 기울이는건 의미 없다. 
=> 가지치기.

***

## 12100. 2048 (Easy)
> https://www.acmicpc.net/problem/12100

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

struct Info {
  int board[20][20];
  void rotate();
  bool up();
};
Info info;
int N;

void init();

int ret;
void dfs(Info& info, int cnt);

int main() {
  init();
  dfs(info, 0);
  cout << ret;
  return 0;
}

void dfs(Info& info, int cnt) {
  if (cnt == 5) {
    for (int i = 0; i < N; ++i) {
      for (int j = 0; j < N; ++j) {
        ret = max(ret, info.board[i][j]);
      }
    }
    return;
  }
  for (int r = 0; r < 4; ++r) {
    Info moved = info;
    if (moved.up()) {
      dfs(moved, cnt + 1);
    }
    info.rotate();
  }
}

bool Info::up() {
  bool moved = false;
  for (int x = 0; x < N; ++x) {
    int top = 0;
    for (int y = 1; y < N; ++y) {
      if (board[y][x] == 0) continue;
      if (board[top][x] == 0) {
        board[top][x] = board[y][x];
        board[y][x] = 0;
        moved = true;
      }
      else if (board[top][x] == board[y][x]) {
        board[top++][x] *= 2;
        board[y][x] = 0;
        moved = true;
      }
      else {
        board[++top][x] = board[y][x];
        if (top != y) {
          board[y][x] = 0;
          moved = true;
        }
      }
    }
  }
  return moved;
}

void Info::rotate() {
  int tmp[20][20];
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      tmp[x][N - 1 - y] = board[y][x];
    }
  }
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      board[y][x] = tmp[y][x];
    }
  }
}

void init() {
  ios::sync_with_stdio(false);
  cin.tie(NULL);
  cin >> N;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      cin >> info.board[i][j];
      ret = max(ret, info.board[i][j]);
    }
  }
}
```
</details>

### 설명
1. 가장 큰 블록만을 출력하면 되는 문제이므로, 보드를 회전하여도 상관 없다. 따라서 보드를 회전시키며 블록을 이동하면, 한 쪽 방향으로 이동하는 부분만 구현하면 된다.
2. info 복사 -> 필요한 작업 수행 -> 재귀 호출 하는것이 더 직관적이다.
3. 보드를 Info struct 에 집어넣고, 보드의 회전, 블록의 이동 연산을 구조체 함수로 구현하면 코드가 깔끔하다.
4. **경우의 수가 많고 복잡한 문제: 각 경우마다 (if문마다)의 예시 손으로 써가면서 확인하기**
   -> 여기서는 if (board[btm][x] == 0), else if (board[btm][x] == board[y][x]), else 각각에서의 예시 써가면서 코드 짜면 편리하다.

***

## 3190. 뱀
> https://www.acmicpc.net/problem/3190

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <deque>
using namespace std;

struct Pos {
  int y, x;
};
struct Info {
  int t;
  char d;
};
const int dy[4] = { 0, 1, 0, -1 }, dx[4] = { 1, 0, -1, 0 };
int N, K, L, y, x, d;
char board[100][100]; // 1: 뱀, 2: 사과
deque<Pos> snake;
deque<Info> dir_info;
void input();

int main() {
  input();
  int t;
  for (t = 1; ; ++t) {
    // 1. 몸길이 늘리기
    y += dy[d]; x += dx[d];
    // 2. 게임 종료 조건
    if (y < 0 || y >= N || x < 0 || x >= N || board[y][x] == 1) break;
    // 3-2. 사과가 없을 때
    if (board[y][x] == 0) {
      board[snake.front().y][snake.front().x] = 0;
      snake.pop_front();
    }
    snake.push_back({ y, x });
    board[y][x] = 1;
    // 4. 방향 변환
    if (!dir_info.empty() && dir_info.front().t == t) {
      if (dir_info.front().d == 'L') {
        d = (d + 3) % 4;
      }
      else {
        d = (d + 1) % 4;
      }
      dir_info.pop_front();
    }
  }
  printf("%d", t);
  return 0;
}

void input() {
  scanf("%d %d", &N, &K);
  while (K--) {
    int i, j;
    scanf("%d %d", &i, &j);
    board[i - 1][j - 1] = 2;
  }
  scanf("%d", &L);
  dir_info.resize(L);
  for (int i = 0; i < L; ++i) {
    scanf("%d %c", &dir_info[i].t, &dir_info[i].d);
  }
  snake.push_back({ 0, 0 });
  board[0][0] = 1;
}
```
</details>

### 설명
deque 활용 문제

***

## 13458: 시험 감독
> https://www.acmicpc.net/problem/13458

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>

int N, A[1000000], B, C;

int main() {
  scanf("%d", &N);
  for (int i = 0; i < N; ++i) {
    scanf("%d", &A[i]);
  }
  scanf("%d %d", &B, &C);

  long long ret = N;
  for (int i = 0; i < N; ++i) {
    if (A[i] > B) {
      A[i] -= B;
      ret += (A[i] + C - 1) / C;
    }
  }

  printf("%lld", ret);

  return 0;
}
```
</details>

### 설명
1. int형 범위를 벗어날 가능성이 있다면 long long 자료형을 사용하기.
2. A를 B로 나눈 후 올림한 값 => (A + B - 1) / B;

***

## 14499. 주사위 굴리기 
> https://www.acmicpc.net/problem/14499

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
using namespace std;

const int dy[4] = { 0, 0, -1, 1 }, dx[4] = { 1, -1, 0, 0 };

int N, M, y, x, K;
int board[20][20];
void init();

struct Dice {
  int up, down, front, right, back, left;
};

Dice roll(Dice& dice, int d) {
  switch (d) {
  case 0: return { dice.left, dice.right, dice.front, dice.up, dice.back, dice.down };
  case 1: return { dice.right, dice.left, dice.front, dice.down, dice.back, dice.up };
  case 2: return { dice.front, dice.back, dice.down, dice.right, dice.up, dice.left };
  case 3: return { dice.back, dice.front, dice.up, dice.right, dice.down, dice.left };
  default: return {};
  }
}

int main() {
  init();
  Dice dice = { 0, 0, 0, 0, 0, 0 };

  for (int i = 0; i < K; ++i) {
    int d; cin >> d; --d;
    int yy = y + dy[d], xx = x + dx[d];
    if (yy < 0 || yy >= N || xx < 0 || xx >= M) continue;
    y = yy; x = xx;
    dice = roll(dice, d);
    if (board[y][x] == 0) {
      board[y][x] = dice.down;
    }
    else {
      dice.down = board[y][x];
      board[y][x] = 0;
    }
    cout << dice.up << '\n';
  }
  return 0;
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N >> M >> y >> x >> K;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < M; ++j) {
      cin >> board[i][j];
    }
  }
}
```
</details>

### 설명
1. 주석을 잘 활용하자.
2. 코드 조금 길어지더라도 안헷갈리게 쓰는게 중요하다.

***

## 14500: 테트로미노
> https://www.acmicpc.net/problem/14500

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

struct Pos {
  int y, x;
  bool operator<(const Pos& rhs) const {
    if (y != rhs.y) return y < rhs.y;
    return x < rhs.x;
  }
  bool operator==(const Pos& rhs) const {
    return y == rhs.y && x == rhs.x;
  }
};

int N, M, board[500][500];
vector<vector<Pos>> shapes;
void init();
vector<Pos> rotate(vector<Pos> p, int num);

int main() {
  init();

  int ret = 0;

  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < M; ++x) {
      for (vector<Pos>& shape : shapes) {
        int tmp = 0;
        for (Pos& pos : shape) {
          int yy = y + pos.y, xx = x + pos.x;
          if (yy < 0 || yy >= N || xx < 0 || xx >= M) {
            tmp = -1;
            break;
          }
          tmp += board[yy][xx];
        }
        ret = max(ret, tmp);
      }
    }
  }

  cout << ret;

  return 0;
}

vector<Pos> rotate(vector<Pos> p, int num) {
  while (num--) {
    vector<Pos> tmp;
    for (Pos& pos : p) {
      tmp.push_back({ pos.x, 3 - pos.y });
    }
    p = tmp;
  }
  return p;
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);
  cin >> N >> M;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < M; ++j) {
      cin >> board[i][j];
    }
  }

  // 기본 도형 + 대칭
  shapes.push_back({ { 0, 0 }, { 0, 1 }, { 0, 2 }, { 0, 3 } });
  shapes.push_back({ {0, 0}, {0, 1}, {1, 0}, {1, 1} });
  shapes.push_back({ { 0, 0 }, { 1, 0 }, { 2, 0 }, { 2, 1 } });
  shapes.push_back({ {0, 0}, {1, 0}, {2, 0}, {2, -1} });
  shapes.push_back({ {0, 0}, {1, 0}, {1, 1}, {2, 1} });
  shapes.push_back({ {0, 0}, {1, -1}, {1, 0}, {2, -1} });
  shapes.push_back({ {0, 0}, {0, 1}, {0, 2}, {1, 1} });

  // 회전 시키면서 삽입 && 왼쪽 위가 (0, 0) 되게 보정
  for (int i = 0; i < 7; ++i) {
    for (int num = 1; num <= 3; ++num) {
      shapes.push_back(rotate(shapes[i], num));
      vector<Pos>& shape = shapes.back();
      sort(shape.begin(), shape.end());
      for (int i = 1; i < shape.size(); ++i) {
        shape[i].y -= shape[0].y;
        shape[i].x -= shape[0].x;
      }
      shape[0] = { 0, 0 };
    }
  }

  sort(shapes.begin(), shapes.end());
  shapes.erase(unique(shapes.begin(), shapes.end()), shapes.end());
}
```
</details>

### 설명
vector<T> 대상으로 emplace_back() 연산을 하면, T() 가 vector의 맨 뒤로 삽입된다. 이를 이용하여 필요할 때마다 vector의 크기를 늘릴 수 있다.

***

## 14501: 퇴사
> https://www.acmicpc.net/problem/14501

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <cstring>
using namespace std;

int N, T[16], P[16], cache[16];
void init();

int maxProfit(int day) { // day일 ~ N일 까지의 최대 수익
  if (day == N + 1) {
    return 0;
  }

  int& ret = cache[day];
  if (ret != -1) return ret;

  ret = maxProfit(day + 1);
  if (T[day] <= N - day + 1) {
    ret = max(ret, maxProfit(day + T[day]) + P[day]);
  }
  return ret;
}

int main() {
  init();
  cout << maxProfit(1);
  return 0;
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N;
  for (int i = 1; i <= N; ++i) {
    cin >> T[i] >> P[i];
  }

  memset(cache, -1, sizeof(cache));
}

/*
#include <cstdio>
#define max(a, b) ((a) > (b) ? (a) : (b))

int N, T[16], P[16];
void input();

int ret;
void dfs(int day, int range, int sum);

int main() {
  input();
  dfs(1, 0, 0);
  printf("%d", ret);
  return 0;
}

void dfs(int day, int range, int sum) {
  if (day == N + 1) {
    ret = max(ret, sum);
    return;
  }
  dfs(day + 1, range, sum);
  if (range < day && day + T[day] - 1 <= N) {
    dfs(day + 1, day + T[day] - 1, sum + P[day]);
  }
}

void input() {
  scanf("%d", &N);
  for (int i = 1; i <= N; ++i) {
    scanf("%d %d", &T[i], &P[i]);
  }
}
*/
```
</details>

### 설명
N값이 최대 15이므로, 모든 경우를 다 세어도 2^15(약 32000) 밖에 되지 않는다.

재귀함수 정의를 어떻게 잡느냐에 따라 코드 복잡도가 크게 달라진다.
***

## 14502. 연구소
> https://www.acmicpc.net/problem/14502

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
#include <algorithm>
using namespace std;

struct Pos {
  int y, x;
};
const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

Pos white[64];
int N, M, w_num;
char board[8][8];
void input();

int ret;
void solve(int w, int idx); // white[w ~ w_num - 1] 까지 3 - idx 선택해서 최대 영역 계산

int main() {
  input();
  solve(0, 3);
  printf("%d", ret);
  return 0;
}

bool discovered[8][8];
int calc() {
  memset(discovered, false, sizeof(discovered));
  queue<Pos> q;

  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < M; ++x) {
      if (board[y][x] == 2) {
        q.push({ y, x });
        discovered[y][x] = true;
      }
    }
  }

  int virus = 0;
  while (!q.empty()) {
    Pos here = q.front(); q.pop();
    for (int d = 0; d < 4; ++d) {
      int yy = here.y + dy[d], xx = here.x + dx[d];
      if (yy < 0 || yy >= N || xx < 0 || xx >= M) continue;
      if (discovered[yy][xx] || board[yy][xx] != 0) continue;
      discovered[yy][xx] = true;
      q.push({ yy, xx });
      ++virus;
    }
  }

  return w_num - 3 - virus;
}

void solve(int w, int num) {
  if (num == 0) {
    ret = max(ret, calc());
    return;
  }
  for (int here = w; here <= w_num - num; ++here) {
    board[white[here].y][white[here].x] = 1;
    solve(here + 1, num - 1);
    board[white[here].y][white[here].x] = 0;
  }
}

void input() {
  scanf("%d %d", &N, &M);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < M; ++j) {
      scanf("%hhd", &board[i][j]);
      if (board[i][j] == 0) {
        white[w_num++] = { i, j };
      }
    }
  }
}
```
</details>

### 설명
1. dfs로 빈칸 선택
2. bfs로 바이러스 확산

***

## 14503. 로봇 청소기
> https://www.acmicpc.net/problem/14503

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>

const int dy[4] = { -1, 0, 1, 0 }, dx[4] = { 0, 1, 0, -1 };
int N, M, y, x, d;
char board[50][50]; // 0 = 청소되지 않은 빈칸. 1 = 벽. 2 = 청소된 빈칸
void input();

int main() {
	input();
	int ret = 0;
	while (true) {
		// 1. 현재 칸이 아직 청소되지 않은 경우
		if (board[y][x] == 0) {
			board[y][x] = 2;
			++ret;
		}
		bool cleaned = false;
		for (int d = 0; d < 4; ++d) {
			int yy = y + dy[d], xx = x + dx[d];
			if (0 <= yy && yy < N && 0 <= xx && xx < M && board[yy][xx] == 0) {
				cleaned = true;
			}
		}
		// 2. 주변 4칸 중 청소되지 않은 빈칸이 없는 경우
		if (!cleaned) {
			int yy = y - dy[d], xx = x - dx[d];
			if (0 <= yy && yy < N && 0 <= xx && xx < M && board[yy][xx] != 1) {
				y = yy; x = xx;
			}
			else {
				break;
			}
		}
		// 3. 주변 4칸 중 청소되지 않은 빈칸이 있는 경우
		else {
			d = (d + 3) % 4;
			int yy = y + dy[d], xx = x + dx[d];
			if (0 <= yy && yy < N && 0 <= xx && xx < M && board[yy][xx] == 0) {
				y = yy; x = xx;
			}
		}
	}
	printf("%d", ret);
	return 0;
}

void input() {
	scanf("%d %d", &N, &M);
	scanf("%d %d %d", &y, &x, &d);
	for (int i = 0; i < N; ++i) {
		for (int j = 0; j < M; ++j) {
			scanf("%hhd", &board[i][j]);
		}
	}
}
```
</details>

### 설명
d = {0(북), 1(동), 2(남), 3(서)}
1. 시계방향 회전 : d = (d + 1) % 4;
2. 반시계방향 회전 : d = (d - 1 + 4) / d = (d + 3) % 4
3. d 방향으로 전진 : y += dy[d], x += dx[d];
4. d 방향으로 후진 : y -= dy[d], x -= dx[d];

***

## 14888. 연산자 끼워넣기
> https://www.acmicpc.net/problem/14888

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

int N, A[11], op[4];
void init();

int minVal = 2e9, maxVal = -2e9;
void calc(int val, int idx); // 지금까지 계산한 값이 val일때 A[idx ~ N - 1] 까지 계산 후 갱신

int main() {
  init();
  calc(A[0], 1);
  cout << maxVal << '\n' << minVal;
  return 0;
}

void calc(int val, int idx) {
  if (idx == N) {
    minVal = min(minVal, val);
    maxVal = max(maxVal, val);
    return;
  }
  if (op[0] > 0) {
    --op[0];
    calc(val + A[idx], idx + 1);
    ++op[0];
  }
  if (op[1] > 0) {
    --op[1];
    calc(val - A[idx], idx + 1);
    ++op[1];
  }
  if (op[2] > 0) {
    --op[2];
    calc(val * A[idx], idx + 1);
    ++op[2];
  }
  if (op[3] > 0) {
    --op[3];
    calc(val / A[idx], idx + 1);
    ++op[3];
  }
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);
  cin >> N;
  for (int i = 0; i < N; ++i) {
    cin >> A[i];
  }
  cin >> op[0] >> op[1] >> op[2] >> op[3];
}
/*
#include <cstdio>
#include <algorithm>
using namespace std;

int N, A[11], op_num[4]; // + - * /
void input();

int chosed[10], max_v = -1000000001, min_v = 1000000001;
void dfs(int idx);

int main() {
	input();
	dfs(0);
	printf("%d\n%d", max_v, min_v);
	return 0;
}

void calc() {
	int ret = A[0];
	for (int i = 0; i < N - 1; ++i) {
		if (chosed[i] == 0) {
			ret += A[i + 1];
		}
		else if (chosed[i] == 1) {
			ret -= A[i + 1];
		}
		else if (chosed[i] == 2) {
			ret *= A[i + 1];
		}
		else {
			ret /= A[i + 1];
		}
	}
	max_v = max(max_v, ret);
	min_v = min(min_v, ret);
}
void dfs(int idx) {
	if (idx == N - 1) {
		calc();
		return;
	}
	for (int i = 0; i < 4; ++i) {
		if (op_num[i] > 0) {
			--op_num[i];
			chosed[idx] = i;
			dfs(idx + 1);
			++op_num[i];
		}
	}
}

void input() {
	scanf("%d", &N);
	for (int i = 0; i < N; ++i) {
		scanf("%d", &A[i]);
	}
	for (int i = 0; i < 4; ++i) {
		scanf("%d", &op_num[i]);
	}
}
*/
```
</details>

### 설명
dfs로 연산자 선택하는 문제.

재귀함수 어떻게 설계하느냐에 따라 코드 복잡도가 크게 달라진다.
***

## 14889. 스타트와 링크
> https://www.acmicpc.net/problem/14889

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <vector>
#include <cmath>
using namespace std;

int N, S[20][20], row[20], col[20];
void init();

int ret = 987654321;
// 선택된 start팀원이 startMask일 때, idx ~ N - 1 사람까지 cnt 명을 선택한 후 계산
void calc(int idx, int cnt, int startMask);

int main() {
  init();
  calc(0, N / 2, 0);
  cout << ret;
  return 0;
}

void calc(int idx, int cnt, int startMask) {
  if (cnt == 0) {
    int gap = 0;
    for (int i = 0; i < N; ++i) {
      if (startMask & (1 << i)) {
        gap += row[i];
      }
      else {
        gap -= col[i];
      }
    }
    ret = min(ret, abs(gap));
    return;
  }

  if (idx < N - cnt) {
    calc(idx + 1, cnt, startMask);
  }
  calc(idx + 1, cnt - 1, startMask | (1 << idx));
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);
  cin >> N;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      cin >> S[i][j];
      row[i] += S[i][j];
      col[j] += S[i][j];
    }
  }
}
```
</details>

### 설명
1. dfs로 팀 선택.
2. 기저사례에서 점수 차 계산하는 문제.

***

## 14890. 경사로
> https://www.acmicpc.net/problem/14890

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <cmath>
using namespace std;

int N, L, H[100][100];
void init();

bool check(int* tmp);

int main() {
  init();

  int ret = 0, tmp[100];
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      tmp[j] = H[i][j];
    }
    if (check(tmp)) {
      ++ret;
    }
  }

  for (int j = 0; j < N; ++j) {
    for (int i = 0; i < N; ++i) {
      tmp[i] = H[i][j];
    }
    if (check(tmp)) {
      ++ret;
    }
  }

  cout << ret;

  return 0;
}

bool check(int* tmp) {
  bool put[100] = { false, };

  for (int x = 0; x < N - 1; ++x) {
    if (tmp[x] == tmp[x + 1]) continue;
    else if (abs(tmp[x] - tmp[x + 1]) > 1) return false;
    else if (tmp[x] == tmp[x + 1] + 1) {
      for (int i = 0; i < L; ++i) {
        if (!(x + 1 + i < N && tmp[x + 1 + i] == tmp[x + 1] && !put[x + 1 + i])) {
          return false;
        }
        put[x + 1 + i] = true;
      }
      x += L - 1;
    }
    else {
      for (int i = 0; i < L; ++i) {
        if (!(x - i >= 0 && tmp[x - i] == tmp[x] && !put[x - i])) {
          return false;
        }
        put[x - i] = true;
      }
    }
  }

  return true;
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N >> L;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      cin >> H[i][j];
    }
  }
}
```
</details>

### 설명
조건을 제대로 읽고 차분히 풀어야 하는 문제

***

## 14891. 톱니바퀴
> https://www.acmicpc.net/problem/14891

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <cstring>
using namespace std;

char state[4][9];
int K;
void init();

bool rotated[4];
void rotate(int idx, int dir);

int main() {
  init();

  while (K--) {
    int idx, dir;
    cin >> idx >> dir;
    --idx;
    memset(rotated, false, sizeof(rotated));
    rotate(idx, dir);
  }

  int ret = 0;
  if (state[0][0] == '1') ret += 1;
  if (state[1][0] == '1') ret += 2;
  if (state[2][0] == '1') ret += 4;
  if (state[3][0] == '1') ret += 8;

  cout << ret;

  return 0;
}

void rotate(int idx, int dir) {
  rotated[idx] = true;

  char tmp[8];
  if (dir == 1) {
    tmp[0] = state[idx][7];
    for (int i = 0; i < 7; ++i) {
      tmp[i + 1] = state[idx][i];
    }
  }
  else {
    tmp[7] = state[idx][0];
    for (int i = 1; i < 8; ++i) {
      tmp[i - 1] = state[idx][i];
    }
  }

  if (idx > 0 && !rotated[idx - 1] && state[idx - 1][2] != state[idx][6]) {
    rotate(idx - 1, dir * -1);
  }
  if (idx < 3 && !rotated[idx + 1] && state[idx + 1][6] != state[idx][2]) {
    rotate(idx + 1, dir * -1);
  }

  memcpy(state[idx], tmp, sizeof(tmp));
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  for (int i = 0; i < 4; ++i) {
    cin >> state[i];
  }
  cin >> K;
}
```
</details>

### 설명
**코드 순서 주의하기**

톱니바퀴 돌리기 전에 재귀호출을 먼저 해야 한다!!

***

## 15683. 감시
> https://www.acmicpc.net/problem/15683

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#define min(a, b) ((a) < (b) ? (a) : (b))

struct Info {
	int y, x, type;
};

const int dy[4] = { -1, 0, 1, 0 }, dx[4] = { 0, 1, 0, -1 }; // 위, 오, 아, 왼

int N, M, c_num;
int d_num[5] = { 4, 2, 4, 4, 1 };
char board[8][8]; // 0: 빈칸, 1~5: cctv, 6: 벽, 7: 감시됨
Info cctv[8];
void input();

int ret = 100;
void dfs(int idx);

int main() {
	input();
	dfs(0);
	printf("%d", ret);
}

void fill_board(int y, int x, int d) {
	while (0 <= y && y < N && 0 <= x && x < M && board[y][x] != 6) {
		board[y][x] = 7;
		y += dy[d]; x += dx[d];
	}
}

void watch(int idx, int dir) {
	Info& c = cctv[idx];
	if (c.type == 0) {
		fill_board(c.y, c.x, dir);
	}
	else if (c.type == 1) {
		fill_board(c.y, c.x, dir);
		fill_board(c.y, c.x, (dir + 2) % 4);
	}
	else if (c.type == 2) {
		fill_board(c.y, c.x, dir);
		fill_board(c.y, c.x, (dir + 1) % 4);
	}
	else if (c.type == 3) {
		for (int d = 0; d < 4; ++d) {
			if (d == dir) continue;
			fill_board(c.y, c.x, d);
		}
	}
	else if (c.type == 4) {
		for (int d = 0; d < 4; ++d) {
			fill_board(c.y, c.x, d);
		}
	}
}

void dfs(int idx) {
	if (idx == c_num) {
		int cnt = 0;
		for (int i = 0; i < N; ++i) {
			for (int j = 0; j < M; ++j) {
				if (board[i][j] == 0) {
					++cnt;
				}
			}
		}
		ret = min(ret, cnt);
		return;
	}

	int tmp[8][8];
	memcpy(tmp, board, sizeof(board));

	for (int d = 0; d < d_num[cctv[idx].type]; ++d) {
		watch(idx, d);
		dfs(idx + 1);
		memcpy(board, tmp, sizeof(board));
	}
}

void input() {
	scanf("%d %d", &N, &M);
	for (int i = 0; i < N; ++i) {
		for (int j = 0; j < M; ++j) {
			scanf("%hhd", &board[i][j]);
			if (1 <= board[i][j] && board[i][j] <= 5) {
				cctv[c_num++] = { i, j, board[i][j] - 1 };
			}
		}
	}
}
```
</details>

### 설명
dfs 함수 내부에서 board에 연산을 하기 전에, board 값을 미리 저장해두고 다음 연산 이전에 복원하는 방식을 사용하면 깔끔하게 코드를 작성할 수 있다.

***

## 15684. 사다리 조작
> https://www.acmicpc.net/problem/15684

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
using namespace std;

int N, M, H;
bool line[31][11];
void init();

bool dfs(int y, int x, int num) { // (y, x) 부터 (H, N -1) 까지 가로선 num개 추가해서 정답 구해지는지 여부 반환
  if (num == 0) {
    for (int x = 1; x <= N; ++x) {
      int pos = x;
      for (int y = 1; y <= H; ++y) {
        if (line[y][pos - 1]) {
          --pos;
        }
        else if (line[y][pos]) {
          ++pos;
        }
      }
      if (pos != x) {
        return false;
      }
    }
    return true;
  }

  if (y == H + 1) {
    return false;
  }
  if (x == N) {
    return dfs(y + 1, 1, num);
  }

  if (line[y][x]) {
    return dfs(y, x + 1, num);
  }

  // 선 연결 하기
  if (!line[y][x - 1]) {
    line[y][x] = true;
    if (dfs(y, x + 1, num - 1)) {
      return true;
    }
    line[y][x] = false;
  }

  // 선 연결 안하기
  return dfs(y, x + 1, num);
}

int main() {
  init();
  for (int num = 0; num <= 3; ++num) {
    if (dfs(1, 1, num)) {
      cout << num;
      return 0;
    }
  }
  cout << -1;
  return 0;
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N >> M >> H;
  for (int i = 0; i < M; ++i) {
    int a, b;
    cin >> a >> b;
    line[a][b] = true;
  }
}
/*
#include <cstdio>
#include <cstring>

int N, M, H, nums[11];
bool hori[31][11];
void input();

int ret = 4;
bool dfs(int cnt, int y, int x);

int main() {
  input();
  for (int num = 0; num <= 3; ++num) {
    if (dfs(num, 1, 1)) {
      printf("%d", num);
      return 0;
    }
  }
  printf("-1");
  return 0;
}

bool calc() {
  for (int x = 1; x <= N; ++x) {
    nums[x] = x;
  }

  for (int x = 1; x <= N - 1; ++x) {
    int xx = x;
    for (int y = 1; y <= H; ++y) {
      if (hori[y][xx - 1]) {
        --xx;
      }
      else if (hori[y][xx]) {
        ++xx;
      }
    }
    if (x != xx) return false;
  }

  return true;
}

bool dfs(int num, int y, int x) {
  if (num == 0) {
    return calc();
  }

  for (int xx = x; xx <= N - 1; ++xx) {
    if (hori[y][xx - 1] || hori[y][xx] || hori[y][xx + 1]) continue;
    hori[y][xx] = true;
    if (dfs(num - 1, y, xx)) return true;
    hori[y][xx] = false;
  }

  for (int yy = y + 1; yy <= H; ++yy) {
    for (int xx = 1; xx <= N - 1; ++xx) {
      if (hori[yy][xx - 1] || hori[yy][xx] || hori[yy][xx + 1]) continue;
      hori[yy][xx] = true;
      if (dfs(num - 1, yy, xx + 1)) return true;
      hori[yy][xx] = false;
    }
  }
  return false;
}

void input() {
  scanf("%d %d %d", &N, &M, &H);
  while (M--) {
    int a, b;
    scanf("%d %d", &a, &b);
    hori[a][b] = true;
  }
}
*/
```
</details>

### 설명
1. 좌 / 우 또는 상 / 하 의 범위를 확인해야 하는 문제는 인덱스를 1부터 저장하면, 범위를 넘어가는 예외를 간단히 처리할 수 있다.
   
   이 문제에서는 hori 배열의 인덱스를 1부터 저장함으로써, 좌 / 우를 확인할 때 범위를 확인하지 않도록 할 수 있다. 

2. 2차원 배열 위에서 dfs를 할 때, (y, x) 값부터 탐색을 하도록 행과 열 값을 입력받도록 하면 순서만 다를 뿐 같은 경우의 수를 중복해서 세지 않을 수 있다.

   NxM board 위에서 dfs를 수행한다고 가정하자.
   
   dfs(y, x) 내부에서 board[y][x ~ M - 1] 탐색 이후, board[y + 1 ~ N - 1][0 ~ M - 1] 을 탐색하도록 구현할 수 있다.

> **dfs 내부에서 선택 하거나 선택하지 않거나로 만들면 내부에 반복문을 만들지 않아도 된다 !!! 훨씬 간단해진다**

***

## 15685: 드래곤 커브
> https://www.acmicpc.net/problem/15685

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

const int dy[4] = { 0, -1, 0, 1 }, dx[4] = { 1, 0, -1, 0 }; // 오, 위, 왼, 아

int N, y, x, d, g;
bool dragon[101][101];

void draw(int y, int x, int d, int g);

int main() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N;
  for (int i = 0; i < N; ++i) {
    cin >> x >> y >> d >> g;
    draw(y, x, d, g);
  }

  int ret = 0;
  for (int y = 0; y < 100; ++y) {
    for (int x = 0; x < 100; ++x) {
      if (dragon[y][x] && dragon[y][x + 1] && dragon[y + 1][x] && dragon[y + 1][x + 1]) {
        ++ret;
      }
    }
  }

  cout << ret;

  return 0;
}

void draw(int y, int x, int d, int g) {
  vector<int> dirs(1, d);
  while (g--) {
    vector<int> tmp = dirs;
    reverse(tmp.begin(), tmp.end());
    for_each(tmp.begin(), tmp.end(), [](int& d) {
      d = (d + 1) % 4;
      });
    dirs.insert(dirs.end(), tmp.begin(), tmp.end());
  }
  dragon[y][x] = true;
  for (int dir : dirs) {
    y += dy[dir]; x += dx[dir];
    dragon[y][x] = true;
  }
}
```
</details>

### 설명
0 <= x, y <= 100 이다. 즉 2차원 배열의 크기는 **101 x 101** 로 해야 한다.

변수 범위를 항상 꼼꼼이 체크해야 한다.

***

## 15686. 치킨 배달
> https://www.acmicpc.net/problem/15686

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
using namespace std;

#define min(a, b) ((a) < (b) ? (a) : (b))

struct Pos {
	int y, x;
};
const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

int N, M, c_num;
Pos chicken[13];
char board[50][50];
void input();

int chosed[13], ret = 987654321;
void dfs(int idx, int from, int num);

int main() {
	input();
	dfs(0, 0, M);
	printf("%d", ret);
	return 0;
}

int dist[50][50];
void calc() {
	memset(dist, -1, sizeof(dist));
	queue<Pos> q;
	for (int i = 0; i < M; ++i) {
		Pos& p = chicken[chosed[i]];
		q.push(p);
		dist[p.y][p.x] = 0;
	}

	int sum = 0;
	while (!q.empty()) {
		Pos here = q.front(); q.pop();
		if (board[here.y][here.x] == 1) {
			sum += dist[here.y][here.x];
		}
		for (int d = 0; d < 4; ++d) {
			int yy = here.y + dy[d], xx = here.x + dx[d];
			if (yy < 0 || yy >= N || xx < 0 || xx >= N || dist[yy][xx] != -1) continue;
			dist[yy][xx] = dist[here.y][here.x] + 1;
			q.push({ yy, xx });
		}
	}

	ret = min(ret, sum);
}

void dfs(int idx, int from, int num) {
	if (num == 0) {
		calc();
		return;
	}
	for (int here = from; here <= c_num - num; ++here) {
		chosed[idx] = here;
		dfs(idx + 1, here + 1, num - 1);
	}
}

void input() {
	scanf("%d %d", &N, &M);
	for (int i = 0; i < N; ++i) {
		for (int j = 0; j < N; ++j) {
			scanf("%hhd", &board[i][j]);
			if (board[i][j] == 2) {
				chicken[c_num++] = { i, j };
			}
		}
	}
}
```
</details>

### 설명
1. dfs로 치킨 집을 선택
2. bfs로 치킨 거리 계산
   2.1. 간선의 가중치가 없을 때의 최단거리는 bfs로 구할 수 있다.
   2.2. 시작 지점이 여러곳인 경우, 해당 시작 지점들을 모두 큐에 넣은 후 bfs를 수행하면 된다.
3. dfs 내부에서 반복문을 쓰지 않고 선택하기 or 선택안하기 로 구현할 수도 있지만 불필요한 재귀도 타게 된다. 
   지금 방식으로 구현하자.

***

## 5373. 큐빙
> https://www.acmicpc.net/problem/5373

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>
#include <cstring>
using namespace std;

enum Face {
	U, D, F, R, B, L
};

int T, n;

struct Cube {
	char color[6][3][3];
	Cube() {
		for (int i = 0; i < 6; ++i) {
			for (int j = 0; j < 3; ++j) {
				for (int k = 0; k < 3; ++k) {
					color[i][j][k] = "wyrbog"[i];
				}
			}
		}
	}
	void rotate(char (*f)[3], bool cw) {
		char tmp[3][3];
		for (int y = 0; y < 3; ++y) {
			for (int x = 0; x < 3; ++x) {
				if (cw) tmp[x][2 - y] = f[y][x];
				else tmp[2 - x][y] = f[y][x];
			}
		}
		memcpy(f, tmp, sizeof(tmp));
	}
	void rotate(char* f0, char* f1, char* f2, char* f3, char* f4, char* f5, char* f6, char* f7, char* f8, char* f9,
		char* f10, char* f11, bool cw) {
		if (cw) {
			char tmp[3] = { *f0, *f1, *f2 };
			*f0 = *f9; *f1 = *f10; *f2 = *f11;
			*f9 = *f6; *f10 = *f7; *f11 = *f8;
			*f6 = *f3; *f7 = *f4; *f8 = *f5;
			*f3 = tmp[0]; *f4 = tmp[1]; *f5 = tmp[2];
		}
		else {
			char tmp[3] = { *f0, *f1, *f2 };
			*f0 = *f3; *f1 = *f4; *f2 = *f5;
			*f3 = *f6; *f4 = *f7; *f5 = *f8;
			*f6 = *f9; *f7 = *f10; *f8 = *f11;
			*f9 = tmp[0]; *f10 = tmp[1]; *f11 = tmp[2];
		}
	}
	void rotate(Face face, bool cw) {
		char tmp[6][3][3];
		memcpy(tmp, color, sizeof(color));

		switch (face) {
		case U:
			rotate(tmp[U], cw);
			rotate(&tmp[F][0][2], &tmp[F][0][1], &tmp[F][0][0],
				&tmp[L][0][2], &tmp[L][0][1], &tmp[L][0][0],
				&tmp[B][0][2], &tmp[B][0][1], &tmp[B][0][0],
				&tmp[R][0][2], &tmp[R][0][1], &tmp[R][0][0],
				cw);
			break;
		case D:
			rotate(tmp[D], !cw);
			rotate(&tmp[F][2][2], &tmp[F][2][1], &tmp[F][2][0],
				&tmp[L][2][2], &tmp[L][2][1], &tmp[L][2][0],
				&tmp[B][2][2], &tmp[B][2][1], &tmp[B][2][0],
				&tmp[R][2][2], &tmp[R][2][1], &tmp[R][2][0],
				!cw);
			break;
		case F:
			rotate(tmp[F], cw);
			rotate(&tmp[U][2][0], &tmp[U][2][1], &tmp[U][2][2],
				&tmp[R][0][0], &tmp[R][1][0], &tmp[R][2][0],
				&tmp[D][2][2], &tmp[D][2][1], &tmp[D][2][0],
				&tmp[L][2][2], &tmp[L][1][2], &tmp[L][0][2],
				cw);
			break;
		case B:
			rotate(tmp[B], cw);
			rotate(&tmp[U][0][0], &tmp[U][0][1], &tmp[U][0][2],
				&tmp[R][0][2], &tmp[R][1][2], &tmp[R][2][2],
				&tmp[D][0][2], &tmp[D][0][1], &tmp[D][0][0],
				&tmp[L][2][0], &tmp[L][1][0], &tmp[L][0][0],
				!cw);
			break;
		case R:
			rotate(tmp[R], cw);
			rotate(&tmp[U][2][2], &tmp[U][1][2], &tmp[U][0][2],
				&tmp[B][0][0], &tmp[B][1][0], &tmp[B][2][0],
				&tmp[D][0][2], &tmp[D][1][2], &tmp[D][2][2],
				&tmp[F][2][2], &tmp[F][1][2], &tmp[F][0][2],
				cw);
			break;
		case L:
			rotate(tmp[L], cw);
			rotate(&tmp[U][2][0], &tmp[U][1][0], &tmp[U][0][0],
				&tmp[B][0][2], &tmp[B][1][2], &tmp[B][2][2],
				&tmp[D][0][0], &tmp[D][1][0], &tmp[D][2][0],
				&tmp[F][2][0], &tmp[F][1][0], &tmp[F][0][0],
				!cw);
			break;
		}
		memcpy(color, tmp, sizeof(color));
	}
};

int main() {
	ios_base::sync_with_stdio(false);
	cin.tie(NULL);

	cin >> T;
	while (T--) {
		Cube cube;
		cin >> n;
		while (n--) {
			string s; cin >> s;
			switch (s[0]) {
			case 'U':
				cube.rotate(U, (s[1] == '+'));
				break;
			case 'D':
				cube.rotate(D, (s[1] == '+'));
				break;
			case 'F':
				cube.rotate(F, (s[1] == '+'));
				break;
			case 'R':
				cube.rotate(R, (s[1] == '+'));
				break;
			case 'B':
				cube.rotate(B, (s[1] == '+'));
				break;
			case 'L':
				cube.rotate(L, (s[1] == '+'));
				break;
			}
		}
		for (int i = 0; i < 3; ++i) {
			for (int j = 0; j < 3; ++j) {
				cout << cube.color[U][i][j];
			}
			cout << '\n';
		}
	}
	
	return 0;
}
```
</details>

### 설명
enum Face { U, D, F, R, B, L }; 을 사용하면 덜 헷갈리게 코드 작성 가능하다.
***

## 16234. 인구 이동
> https://www.acmicpc.net/problem/16234

### 코드
<details>
<summary>C++</summary>

```cpp
// DFS
#include <iostream>
#include <queue>
#include <vector>
#include <cmath>
#include <cstring>
using namespace std;

struct Pos {
  int y, x;
};

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

int N, L, R, A[50][50];
void init();

vector<vector<bool>> visited;
vector<Pos> nations;
queue<Pos> changed;
int dfs(Pos here);
bool p_move();

int main() {
  init();
  int day;
  for (day = 0; ; ++day) {
    if (!p_move()) {
      break;
    }
  }
  cout << day;
  return 0;
}

int dfs(Pos here) {
  visited[here.y][here.x] = true;
  nations.push_back(here);
  int num = A[here.y][here.x];

  for (int d = 0; d < 4; ++d) {
    Pos there = { here.y + dy[d], here.x + dx[d] };
    if (there.y < 0 || there.y >= N || there.x < 0 || there.x >= N || visited[there.y][there.x]) {
      continue;
    }
    if (abs(A[there.y][there.x] - A[here.y][here.x]) < L || abs(A[there.y][there.x] - A[here.y][here.x]) > R) {
      continue;
    }
    num += dfs(there);
  }

  return num;
}

bool p_move() {
  bool opened = false;
  visited = vector<vector<bool>>(N, vector<bool>(N, false));

  int _size = changed.size();
  for (int i = 0; i < _size; ++i) {
    Pos p = changed.front(); changed.pop();
    if (!visited[p.y][p.x]) {
      nations.clear();
      int total = dfs({ p.y, p.x });
      if (nations.size() > 1) {
        opened = true;
        int avg = total / nations.size();
        for (Pos& nation : nations) {
          A[nation.y][nation.x] = avg;
          changed.push(nation);
        }
      }
    }
  }

  return opened;
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N >> L >> R;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      cin >> A[i][j];
      changed.push({ i, j });
    }
  }
}

// BFS
#include <iostream>
#include <queue>
#include <vector>
#include <cmath>
#include <cstring>
using namespace std;

struct Pos {
  int y, x;
};

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

int N, L, R, A[50][50];
void init();

queue<Pos> changed;
vector<vector<bool>> discovered;
bool bfs();
void bfs(Pos here);

int main() {
  init();
  int day;
  for (day = 0; ; ++day) {
    if (!bfs()) {
      break;
    }
  }
  cout << day;
  return 0;
}

bool bfs() {
  int num = changed.size();
  discovered = vector<vector<bool>>(N, vector<bool>(N, false));
  for (int i = 0; i < num; ++i) {
    Pos p = changed.front(); changed.pop();
    if (!discovered[p.y][p.x]) {
      bfs(p);
    }
  }
  return !changed.empty();
}

void bfs(Pos start) {
  queue<Pos> q;
  vector<Pos> moved;
  int sum = 0;

  q.push(start);
  discovered[start.y][start.x] = true;

  while (!q.empty()) {
    Pos here = q.front(); q.pop();
    moved.push_back(here);
    sum += A[here.y][here.x];
    for (int d = 0; d < 4; ++d) {
      Pos there = { here.y + dy[d], here.x + dx[d] };
      if (there.y < 0 || there.y >= N || there.x < 0 || there.x >= N || discovered[there.y][there.x]) continue;
      if (abs(A[there.y][there.x] - A[here.y][here.x]) < L || abs(A[there.y][there.x] - A[here.y][here.x]) > R) continue;
      discovered[there.y][there.x] = true;
      q.push(there);
    }
  }

  int avg = sum / moved.size();
  if (moved.size() > 1) {
    for (Pos& p : moved) {
      A[p.y][p.x] = avg;
      changed.push(p);
    }
  }
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N >> L >> R;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      cin >> A[i][j];
      changed.push({ i, j });
    }
  }
}
```
</details>

### 설명
1. dfs 또는 bfs를 이용하여 문제를 해결할 수 있다.

2. queue를 이용하면 이전 반복에서 변화가 있었던 좌표 대상으로만 수행할 수 있다.

   (변화 x - 변화 x) 국경선은 열릴 수 없다. 즉 이전 반복에서 변화가 있었던 나라들을 대상으로만 탐색을 진행하면 된다.

   둘 다 변화 없었다면, 둘 사이의 국경선도 전날 열리지 않았다는 것이고, 그럼 둘 사이의 범위는 [L, R] 이 아니라는 것이고, 그럼 오늘도 국경은 안열린다.

***

## 16235. 나무 재테크
> https://www.acmicpc.net/problem/16235

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <deque>
#include <algorithm>
using namespace std;

int N, M, K, A[10][10];
void init();

int food[10][10];
deque<int> trees[10][10];

int main() {
	init();

	for (int year = 0; year < K; ++year) {
		// 1. 봄 && 여름
		for (int y = 0; y < N; ++y) {
			for (int x = 0; x < N; ++x) {
				for (int i = 0; i < trees[y][x].size(); ++i) {
					int& age = trees[y][x][i];
					if (food[y][x] >= age) {
						food[y][x] -= age;
						++age;
					}
					else {
						for (int j = i; j < trees[y][x].size(); ++j) {
							food[y][x] += trees[y][x][j] / 2;
						}
						trees[y][x].erase(trees[y][x].begin() + i, trees[y][x].end());
					}
				}
			}
		}

		// 2. 가을
		for (int y = 0; y < N; ++y) {
			for (int x = 0; x < N; ++x) {
				for (int age : trees[y][x]) {
					if (age % 5 == 0) {
						for (int i = -1; i <= 1; ++i) {
							for (int j = -1; j <= 1; ++j) {
								if (i == 0 && j == 0) continue;
								int yy = y + i, xx = x + j;
								if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
								trees[yy][xx].push_front(1);
							}
						}
					}
				}
			}
		}

		// 3. 겨울
		for (int y = 0; y < N; ++y) {
			for (int x = 0; x < N; ++x) {
				food[y][x] += A[y][x];
			}
		}
	}

	int ret = 0;
	for (int y = 0; y < N; ++y) {
		for (int x = 0; x < N; ++x) {
			ret += trees[y][x].size();
		}
	}
	cout << ret;

	return 0;
}

void init() {
	ios_base::sync_with_stdio(false);
	cin.tie(NULL);

	cin >> N >> M >> K;
	for (int i = 0; i < N; ++i) {
		for (int j = 0; j < N; ++j) {
			cin >> A[i][j];
		}
	}

	for (int i = 0; i < M; ++i) {
		int y, x, z;
		cin >> y >> x >> z;
		--y; --x;
		trees[y][x].push_back(z);
	}

	for (int i = 0; i < N; ++i) {
		for (int j = 0; j < N; ++j) {
			sort(trees[i][j].begin(), trees[i][j].end());
			food[i][j] = 5;
		}
	}
}
```
</details>

### 설명
데크 활용 문제.

***

## 16236. 아기 상어
> https://www.acmicpc.net/problem/16236

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <queue>
#include <vector>
#include <algorithm>
using namespace std;

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

struct Pos {
  int y, x;
  bool operator<(const Pos& rhs) const {
    if (y != rhs.y) return y < rhs.y;
    return x < rhs.x;
  }
};

struct Shark {
  Pos pos;
  int size, cnt;
};

int N, board[20][20];
Shark shark;
void init();

Pos cand;
vector<vector<int>> dist;
bool choose();

int main() {
  init();

  int t = 0;
  while (true) {
    // 1. 먹을 수 있는 물고기 후보 선택
    if (!choose()) {
      break;
    }
    // 2. 해당 물고기로 이동
    shark.pos = cand;
    // 3. 물고기 먹기
    board[cand.y][cand.x] = 0;
    --shark.cnt;
    // 4. 크기 증가
    if (shark.cnt == 0) {
      ++shark.size;
      shark.cnt = shark.size;
    }
    // 5. 시간 늘리기
    t += dist[cand.y][cand.x];
  }

  cout << t;

  return 0;
}

bool choose() {
  queue<Pos> q;
  dist = vector<vector<int>>(N, vector<int>(N, -1));
  vector<Pos> cands;
  int minDist = -1;

  q.push(shark.pos);
  dist[shark.pos.y][shark.pos.x] = 0;

  while (!q.empty()) {
    Pos here = q.front(); q.pop();
    if (minDist != -1 && dist[here.y][here.x] >= minDist) {
      break;
    }
    for (int d = 0; d < 4; ++d) {
      Pos there = { here.y + dy[d], here.x + dx[d] };
      if (there.y < 0 || there.y >= N || there.x < 0 || there.x >= N || dist[there.y][there.x] != -1) continue;
      if (board[there.y][there.x] > shark.size) continue;
      dist[there.y][there.x] = dist[here.y][here.x] + 1;
      q.push(there);
      if (board[there.y][there.x] == 0) continue;
      if (board[there.y][there.x] < shark.size) {
        minDist = dist[there.y][there.x];
        cands.push_back(there);
      }
    }
  }

  if (cands.empty()) return false;

  sort(cands.begin(), cands.end());
  cand = cands[0];
  return true;
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      cin >> board[i][j];
      if (board[i][j] == 9) {
        board[i][j] = 0;
        shark.pos = { i, j };
        shark.size = 2;
        shark.cnt = 2;
      }
    }
  }
}
```
</details>

### 설명
bfs 구현 문제.

bfs 탐색 시 거리가 가까운 정점부터 순차적으로 탐색된다는 것을 이용하면 코드를 간략하게 짤 수 있다.

상어가 먹을 수 있는 물고기가 발견되는 순간의 거리가 최단거리가 됨을 알 수 있고, 해당 최단거리를 갖는 정점을 큐에서 꺼낸 이후부터는 탐색을 더 할 필요가 없다. (해당 최단 거리 + 1 인 정점을 탐색하므로)
***

## 17144. 미세먼지 안녕!
> https://www.acmicpc.net/problem/17144

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

struct Pos {
  int y, x;
};

int R, C, T, dust[50][50];
Pos air = { -1, -1 };
void init();

void spread();
void cleaning();

int main() {
  init();

  for (int t = 0; t < T; ++t) {
    // 1. 확산
    spread();
    // 2. 공기청정기 작동
    cleaning();
  }

  int sum = 0;
  for (int y = 0; y < R; ++y) {
    for (int x = 0; x < C; ++x) {
      if (dust[y][x] == -1)continue;
      sum += dust[y][x];
    }
  }
  cout << sum;

  return 0;
}

void cleaning() {
  // 1. 반시계
  int y = air.y - 1, x = air.x;
  while (y > 0) {
    dust[y][x] = dust[y - 1][x];
    --y;
  }
  while (x < C - 1) {
    dust[y][x] = dust[y][x + 1];
    ++x;
  }
  while (y < air.y) {
    dust[y][x] = dust[y + 1][x];
    ++y;
  }
  while (x > 1) {
    dust[y][x] = dust[y][x - 1];
    --x;
  }
  dust[air.y][air.x + 1] = 0;
  // 2. 시계
  y = air.y + 2, x = air.x;
  while (y < R - 1) {
    dust[y][x] = dust[y + 1][x];
    ++y;
  }
  while (x < C - 1) {
    dust[y][x] = dust[y][x + 1];
    ++x;
  }
  while (y > air.y + 1) {
    dust[y][x] = dust[y - 1][x];
    --y;
  }
  while (x > 1) {
    dust[y][x] = dust[y][x - 1];
    --x;
  }
  dust[air.y + 1][air.x + 1] = 0;
}

void spread() {
  int tmp[50][50] = { 0, };
  for (int y = 0; y < R; ++y) {
    for (int x = 0; x < C; ++x) {
      if (dust[y][x] <= 0) continue;
      int sum = 0, amount = dust[y][x] / 5;
      for (int d = 0; d < 4; ++d) {
        int yy = y + dy[d], xx = x + dx[d];
        if (yy < 0 || yy >= R || xx < 0 || xx >= C || dust[yy][xx] == -1) continue;
        tmp[yy][xx] += amount;
        sum += amount;
      }
      tmp[y][x] += dust[y][x] - sum;
    }
  }
  tmp[air.y][air.x] = -1;
  tmp[air.y + 1][air.x] = -1;
  memcpy(dust, tmp, sizeof(dust));
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> R >> C >> T;
  for (int i = 0; i < R; ++i) {
    for (int j = 0; j < C; ++j) {
      cin >> dust[i][j];
      if (dust[i][j] == -1 && air.y == -1) {
        air = { i, j };
      }
    }
  }
}
```
</details>

### 설명
구현문제.
board의 변화값을 tmp에 저장하고, 다시 board에 복사하는 경우 

board의 기본값이 tmp에 정확히 저장되는지 확인해야 한다.

여기서는 공기청정기의 위치가 -1로 저장되므로, tmp에도 공기청정기의 위치를 표시해주어야 한다.
***

## 17143. 낚시왕
> https://www.acmicpc.net/problem/17143

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <vector>
#include <cstring>
using namespace std;

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, 1, -1 };

struct Shark {
  int y, x, speed, dir, size;
  bool out;
};

int R, C, M, sharkIdx[100][100];
vector<Shark> sharks;
void init();

int fishing(int c);
void moving(Shark& shark, int dist);
void update(int i);

int main() {
  init();

  int ret = 0;
  for (int c = 0; c < C; ++c) {
    // 1. 낚시
    ret += fishing(c);
    // 2. 상어 이동
    for (Shark& shark : sharks) {
      if (shark.out) continue;
      moving(shark, shark.speed);
    }
    // 3. sharkIdx 갱신
    memset(sharkIdx, -1, sizeof(sharkIdx));
    for (int i = 0; i < sharks.size(); ++i) {
      if (sharks[i].out) continue;
      update(i);
    }
  }

  cout << ret;

  return 0;
}

void update(int i) {
  Shark& shark = sharks[i]; // 새롭게 고려하는 상어
  int& idx = sharkIdx[shark.y][shark.x]; // 기존에 있던 상어 인덱스
  if (idx == -1) {
    idx = i;
  }
  else if (sharks[idx].size > shark.size) {
    shark.out = true;
  }
  else {
    sharks[idx].out = true;
    idx = i;
  }
}

void moving(Shark& shark, int dist) {
  // 상하
  if (shark.dir <= 1) {
    int yy = shark.y + dy[shark.dir] * dist;
    // 위로 범위 넘음
    if (yy < 0) {
      dist -= shark.y;
      shark.y = 0;
      shark.dir = 1;
      moving(shark, dist);
    }
    // 아래로 범위 넘음
    else if (yy >= R) {
      dist -= (R - 1 - shark.y);
      shark.y = R - 1;
      shark.dir = 0;
      moving(shark, dist);
    }
    else {
      shark.y = yy;
    }
  }
  // 좌우
  else {
    int xx = shark.x + dx[shark.dir] * dist;
    // 좌로 범위 넘음
    if (xx < 0) {
      dist -= shark.x;
      shark.x = 0;
      shark.dir = 2;
      moving(shark, dist);
    }
    // 우로 범위 넘음
    else if (xx >= C) {
      dist -= (C - 1 - shark.x);
      shark.x = C - 1;
      shark.dir = 3;
      moving(shark, dist);
    }
    else {
      shark.x = xx;
    }
  }
}

int fishing(int c) {
  for (int r = 0; r < R; ++r) {
    if (sharkIdx[r][c] != -1) {
      Shark& shark = sharks[sharkIdx[r][c]];
      shark.out = true;
      sharkIdx[r][c] = -1;
      return shark.size;
    }
  }
  return 0;
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  memset(sharkIdx, -1, sizeof(sharkIdx));

  cin >> R >> C >> M;
  for (int i = 0; i < M; ++i) {
    int r, c, s, d, z;
    cin >> r >> c >> s >> d >> z;
    --r; --c; --d;
    if (d <= 1) s %= (2 * R - 2);
    else s %= (2 * C - 2);
    sharks.push_back({ r, c, s, d, z, false });
    sharkIdx[r][c] = i;
  }
}
```
</details>

### 설명
구현 문제.

1. 최대 100번의 반복을 하므로, 첫 상어 개수를 매 반복마다 순회한다고 해도 최대 O(100 x 100 x 100) = O(1,000,000) 이므로 시간 안에 수행 가능하다.

2. 따라서 Info 구조체 안에 bool out 변수를 사용하여 구현해도 된다는 것을 알 수 있다. 

3. 문제에 따라 아무 상어도 먹히지 않는 입력이 주어질 수 있다. 따라서 상어가 먹힘으로써 줄어듦을 이용하여 시간복잡도를 줄일려는 노력은 의미 없다.

***

## 17140. 이차원 배열과 연산
> https://www.acmicpc.net/problem/17140

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct Info {
  int n, c;
  bool operator<(const Info& rhs) const {
    if (c != rhs.c) return c < rhs.c;
    return n < rhs.n;
  }
};

int r, c, k;
int rowNum = 3, colNum = 3;
vector<vector<int>> A(100, vector<int>(100, 0));
void revert();
void init();

int sortRow();

int main() {
  init();

  if (A[r][c] == k) {
    cout << 0;
    return 0;
  }

  for (int t = 1; t <= 100; ++t) {

    bool R = rowNum >= colNum;

    if (R) {
      colNum = sortRow();
    }
    else {
      revert();
      colNum = sortRow();
      revert();
    }

    if (A[r][c] == k) {
      cout << t;
      return 0;
    }
  }

  cout << -1;
  return 0;
}

int sortRow() {
  int ret = 0;
  for (int y = 0; y < rowNum; ++y) {
    // 1. 숫자 카운트
    vector<Info> infos;
    int cnt[101] = { 0, };
    for (int x = 0; x < colNum; ++x) {
      if (A[y][x] > 0) {
        ++cnt[A[y][x]];
      }
    }
    // 2. (숫자, 개수) 추가 및 정렬
    for (int n = 1; n <= 100; ++n) {
      if (cnt[n] == 0) continue;
      infos.push_back({ n, cnt[n] });
    }
    sort(infos.begin(), infos.end());
    // 3. (숫자, 개수) 배열에 반영
    A[y] = vector<int>(100, 0);
    int idx = 0;
    for (Info& info : infos) {
      A[y][idx] = info.n;
      A[y][idx + 1] = info.c;
      idx += 2;
      if (idx >= 100) break;
    }
    ret = max(ret, idx);
  }
  return ret;
}

void revert() {
  for (int y = 0; y < 100; ++y) {
    for (int x = y + 1; x < 100; ++x) {
      swap(A[y][x], A[x][y]);
    }
  }
  swap(rowNum, colNum);
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> r >> c >> k;
  --r; --c;
  for (int i = 0; i < 3; ++i) {
    for (int j = 0; j < 3; ++j) {
      cin >> A[i][j];
    }
  }
}
```
</details>

### 설명
1. 반복문을 돌리기 전, 최초에 A[r][c] == k 인 경우를 빼먹으면 안된다.
2. 쉬워보여도 꼭 기능별로 함수를 만들자.

***

## 17142. 연구소 3
> https://www.acmicpc.net/problem/17142

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <queue>
using namespace std;

#define INF 987654321

struct Pos {
  int y, x;
};

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

int N, M, blank, board[50][50];
vector<Pos> viruses;
void init();

int chosed[10], ret = INF;
int bfs();
void calc(int idx, int from, int num);

int main() {
  init();
  calc(0, 0, M);
  cout << (ret == INF ? -1 : ret);
  return 0;
}

int bfs() {
  queue<Pos> q;
  vector<vector<int>> dist(N, vector<int>(N, -1));
  int maxDist = 0, cnt = 0;

  for (int i = 0; i < M; ++i) {
    q.push({ viruses[chosed[i]] });
    dist[viruses[chosed[i]].y][viruses[chosed[i]].x] = 0;
  }
  while (!q.empty()) {
    Pos here = q.front(); q.pop();
    if (board[here.y][here.x] == 0) {
      ++cnt;
      maxDist = max(maxDist, dist[here.y][here.x]);
    }
    for (int d = 0; d < 4; ++d) {
      Pos there = { here.y + dy[d], here.x + dx[d] };
      if (there.y < 0 || there.y >= N || there.x < 0 || there.x >= N) continue;
      if (dist[there.y][there.x] != -1 || board[there.y][there.x] == 1) continue;
      q.push(there);
      dist[there.y][there.x] = dist[here.y][here.x] + 1;
    }
  }

  if (cnt != blank) return INF;

  return maxDist;
}

void calc(int idx, int from, int num) {
  if (num == 0) {
    ret = min(ret, bfs());
    return;
  }
  for (int here = from; here <= viruses.size() - num; ++here) {
    chosed[idx] = here;
    calc(idx + 1, here + 1, num - 1);
  }
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N >> M;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      cin >> board[i][j];
      if (board[i][j] == 2) {
        viruses.push_back({ i, j });
      }
      else if (board[i][j] == 0) {
        ++blank;
      }
    }
  }
}
```
</details>

### 설명
1. dfs 로 퍼트릴 바이러스 선택

2. bfs로 바이러스 퍼트리기

3. bfs 시작 지점이 여러개일 경우, 큐에 모두 집어 넣고 시작하면 된다.

***

## 17779. 게리맨더링 2
> https://www.acmicpc.net/problem/17779

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

int N, A[21][21], total;
bool line[21][21];
void input();

int min_gap = 987654321;
void make_line(int x, int y, int d1, int d2);
void calc(int x, int y, int d1, int d2);

int main() {
  input();
  for (int x = 1; x <= N - 2; ++x) {
    for (int y = 2; y <= N - 1; ++y) {
      for (int d1 = 1; d1 <= N - 2; ++d1) {
        for (int d2 = 1; d2 <= N - 2; ++d2) {
          if (x + d1 + d2 > N) continue;
          if (y - d1 < 1 || y + d2 > N) continue;
          make_line(x, y, d1, d2);
          calc(x, y, d1, d2);
        }
      }
    }
  }
  printf("%d", min_gap);
}

void calc(int x, int y, int d1, int d2) {
  int num[5] = { 0, };

  for (int yy = 1; yy <= y; ++yy) {
    for (int xx = 1; xx < x + d1 && !line[xx][yy]; ++xx) {
      num[0] += A[xx][yy];
    }
  }

  for (int yy = y + 1; yy <= N; ++yy) {
    for (int xx = 1; xx <= x + d2 && !line[xx][yy]; ++xx) {
      num[1] += A[xx][yy];
    }
  }

  for (int yy = 1; yy < y - d1 + d2; ++yy) {
    for (int xx = N; xx >= x + d1 && !line[xx][yy]; --xx) {
      num[2] += A[xx][yy];
    }
  }

  for (int yy = y - d1 + d2; yy <= N; ++yy) {
    for (int xx = N; xx > x + d2 && !line[xx][yy]; --xx) {
      num[3] += A[xx][yy];
    }
  }

  num[4] = total;
  for (int i = 0; i < 4; ++i) {
    num[4] -= num[i];
  }

  sort(num, num + 5);
  min_gap = min(min_gap, num[4] - num[0]);
}

void make_line(int x, int y, int d1, int d2) {
  memset(line, false, sizeof(line));

  int xx = x, yy = y;
  while (xx <= x + d1) {
    line[xx][yy] = true;
    ++xx; --yy;
  }

  xx = x, yy = y;
  while (xx <= x + d2) {
    line[xx][yy] = true;
    ++xx; ++yy;
  }

  xx = x + d1, yy = y - d1;
  while (xx <= x + d1 + d2) {
    line[xx][yy] = true;
    ++xx; ++yy;
  }

  xx = x + d2, yy = y + d2;
  while (xx <= x + d2 + d1) {
    line[xx][yy] = true;
    ++xx; --yy;
  }
}

void input() {
  scanf("%d", &N);
  for (int i = 1; i <= N; ++i) {
    for (int j = 1; j <= N; ++j) {
      scanf("%d", &A[i][j]);
      total += A[i][j];
    }
  }
}
```
</details>

### 설명
가능한 모든 x, y, d1, d2 조합의 수는 O(20 * 20 * 20 * 20) = O(160,000) 으로, 매 반복마다 O(400) 연산을 하여도, 최대 O(64,000,000) 으로 브루트 포스로 풀더라도 시간 내에 충분히 가능하다.

***

## 17837. 새로운 게임 2
> https://www.acmicpc.net/problem/17837

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct Horse { int y, x, d, idx; };

const int dy[4] = { 0, 0, -1, 1 }, dx[4] = { 1, -1, 0, 0 };

int N, K, board[12][12]; // 0: 흰, 1: 빨, 2:파
vector<Horse> horses;
vector<int> horseIdxs[12][12];
void init();

bool moveHorse(Horse& h);

int main() {
  init();

  for (int turn = 1; turn <= 1000; ++turn) {
    bool moreThan4 = false;
    for (int i = 0; i < horses.size(); ++i) {
      Horse& h = horses[i];
      moreThan4 = moveHorse(h);
      if (moreThan4) {
        cout << turn;
        return 0;
      }
    }
  }

  cout << "-1";
  return 0;
}

bool moveHorse(Horse& h) {
  int y = h.y, x = h.x, hIdx = h.idx;
  int yy = h.y + dy[h.d], xx = h.x + dx[h.d];
  // 벗어나는 경우거나 파란색인 경우 예외처리
  if (yy < 0 || yy >= N || xx < 0 || xx >= N || board[yy][xx] == 2) {
    h.d = (h.d <= 1 ? 1 - h.d : 5 - h.d);
    yy = h.y + dy[h.d]; xx = h.x + dx[h.d];
    // 반대로 바꾼 후에도 파란색이거나 벗어나는 경우 종료
    if (yy < 0 || yy >= N || xx < 0 || xx >= N || board[yy][xx] == 2) {
      return false;
    }
  }

  // 1. horses 배열 갱신
  for (int idx = hIdx; idx < horseIdxs[y][x].size(); ++idx) {
    Horse& horse = horses[horseIdxs[y][x][idx]];
    horse.y = yy; horse.x = xx; horse.idx = horseIdxs[yy][xx].size() + idx - hIdx;
  }

  // 2. horseIdxs 배열 갱신
  horseIdxs[yy][xx].insert(horseIdxs[yy][xx].end(), horseIdxs[y][x].begin() + hIdx, horseIdxs[y][x].end());
  horseIdxs[y][x].erase(horseIdxs[y][x].begin() + hIdx, horseIdxs[y][x].end());

  // 3. 빨간색인 경우 순서 바꾸기
  if (board[yy][xx] == 1) {
    reverse(horseIdxs[yy][xx].begin() + h.idx, horseIdxs[yy][xx].end());
    for (int idx = h.idx; idx < horseIdxs[yy][xx].size(); ++idx) {
      Horse& horse = horses[horseIdxs[yy][xx][idx]];
      horse.idx = idx;
    }
  }

  return horseIdxs[yy][xx].size() >= 4;
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N >> K;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      cin >> board[y][x];
    }
  }

  for (int i = 0; i < K; ++i) {
    int y, x, d;
    cin >> y >> x >> d;
    --y; --x; --d;
    horses.push_back({ y, x, d, 0 });
    horseIdxs[y][x].push_back(i);
  }
}
```
</details>

### 설명
1. 말은 번호 순서대로 움직이므로 말들의 정보를 vector에 일렬로 저장한다.

2. 보드를 나타내는 2차원 배열 위에서는, 해당 칸에 존재하는 말들의 vector 상의 인덱스를 저장한다.

3. 참조형 변수를 사용할 때는 이 값이 변경된 값일 수도 있음을 꼭 생각해야 한다.

```cpp
Horse& horse = horses[horseIdxs[y][x][idx]];
horse.y = yy; horse.x = xx; horse.idx = horseIdxs[yy][xx].size() + idx - hIdx;
```
위 코드에서 y, x, hIdx를 각각 h.y, h.x, h.idx 라고 적어 오류가 생겼었다.
h.y, h.x, h.idx는 해당 코드에서 이동되는 좌표를 기준으로 변한다.
따라서 h.y, h.x, h.idx 가 이동되기 전의 좌표를 기준한 값이라고 생각하면 안된다.

이동하기 전 -> 이동 후 로직에서는 각각의 좌표를 꼭 변수에 담아놓자.

***

## 17822. 원판 돌리기
> https://www.acmicpc.net/problem/17822

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <cstring>
using namespace std;

struct Pos {
  int y, x;
};

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

int N, M, T, sum, cnt;
int board[50][50];
void init();

void rotate(int num, int dir, int k);

bool discovered[50][50];
bool bfs(int y, int x);

void print() {
  cout << endl;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < M; ++x) {
      cout << board[y][x] << ' ';
    }
    cout << endl;
  }
}

int main() {
  init();

  while (T--) {
    // 1. 원판 회전
    int x, d, k;
    cin >> x >> d >> k;
    for (int num = x; num <= N; num += x) {
      rotate(num - 1, (d == 0 ? 1 : -1), k);
    }

    // 2. 인접한 수 찾아서 지우기
    bool erased = false;
    memset(discovered, false, sizeof(discovered));
    for (int y = 0; y < N; ++y) {
      for (int x = 0; x < M; ++x) {
        if (!discovered[y][x] && board[y][x] != 0) {
          if (bfs(y, x)) erased = true;
        }
      }
    }

    if (erased) continue;

    // 3. 지워진 수 없으면 평균 구해서 빼고 더하는 과정 수행
    double avg = (double)sum / cnt;
    for (int y = 0; y < N; ++y) {
      for (int x = 0; x < M; ++x) {
        if (board[y][x] == 0) continue;
        if (board[y][x] < avg) {
          ++board[y][x];
          ++sum;
        }
        else if (board[y][x] > avg) {
          --board[y][x];
          --sum;
        }
      }
    }
  }

  cout << sum;

  return 0;
}

bool bfs(int y, int x) {
  queue<Pos> q;
  q.push({ y, x });
  discovered[y][x] = true;
  vector<Pos> visited;

  while (!q.empty()) {
    Pos here = q.front(); q.pop();
    visited.push_back(here);
    for (int d = 0; d < 4; ++d) {
      Pos there = { here.y + dy[d], (here.x + dx[d] + M) % M };
      if (there.y < 0 || there.y >= N) continue;
      if (discovered[there.y][there.x] || board[there.y][there.x] != board[here.y][here.x]) continue;
      discovered[there.y][there.x] = true;
      q.push({ there.y, there.x });
    }
  }
  if (visited.size() == 1) return false;

  for (Pos& pos : visited) {
    sum -= board[pos.y][pos.x];
    board[pos.y][pos.x] = 0;
  }
  cnt -= visited.size();
  return true;
}

void rotate(int num, int dir, int k) {
  int tmp[50];
  for (int c = 0; c < M; ++c) {
    tmp[(c + dir * k + M) % M] = board[num][c];
  }
  memcpy(board[num], tmp, sizeof(tmp));
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N >> M >> T;
  cnt = N * M;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < M; ++j) {
      cin >> board[i][j];
      sum += board[i][j];
    }
  }
}
```
</details>

### 설명
구현 + bfs 문제.

지운 수를 0으로 정의하였으므로, 값이 0인 부분에서 bfs를 수행하지 않도록 해야한다.

bfs / dfs 에서 다음 노드로 탐색을 파고 들어갈 때, 탐색 조건을 만족하는지 꼭 확인해야한다.

문제 조건 잘 확인하기. 평균보다 '큰' 수 -1, '작은' 수 +1 이므로 같은 수에 대해서는 아무 연산이 일어나지 않는다.

**|| 연산자를 사용하면, 왼쪽이 참일 경우 오른쪽은 수행하지 않는다.**

```cpp
if (bfs(num, i)) {
  check = true;
}

check = check || bfs(num, i); // 이렇게 하면 안됨.
```

***

## 17825. 주사위 윷놀이
> https://www.acmicpc.net/problem/17825

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>

#define max(a, b) ((a) > (b) ? (a) : (b))

int score[33] = { 0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30,
                 32, 34, 36, 38, 40, 13, 16, 19, 25, 30, 35, 22, 24, 28, 27, 26, 0 };
int next_pos[33][2] = { {1}, {2}, {3}, {4}, {5}, {6, 21}, {7}, {8}, {9}, {10},
                       {11, 27}, {12}, {13}, {14}, {15}, {16, 29}, {17},
                       {18}, {19}, {20}, {32}, {22}, {23}, {24}, {25},
                       {26}, {20}, {28}, {24}, {30}, {31}, {24}, {0} };
int h_pos[4], dice[10];

int max_score;
void dfs(int turn, int sum);

int main() {
    for (int i = 0; i < 10; ++i) {
        scanf("%d", &dice[i]);
    }
    dfs(0, 0);
    printf("%d", max_score);
    return 0;
}

int get_there(int pos, int d) {
    while (d--) {
        if (pos == 32) break;
        pos = next_pos[pos][0];
    }
    return pos;
}

bool can_go(int there) {
    for (int i = 0; i < 4; ++i) {
        if (h_pos[i] != 32 && h_pos[i] == there) {
            return false;
        }
    }
    return true;
}

void dfs(int turn, int sum) {
    if (turn == 10) {
        max_score = max(max_score, sum);
        return;
    }

    for (int h = 0; h < 4; ++h) {
        int here = h_pos[h], there;
        if (here == 32) continue;
        if (next_pos[here][1] == 0) {
            there = get_there(next_pos[here][0], dice[turn] - 1);
        }
        else {
            there = get_there(next_pos[here][1], dice[turn] - 1);
        }
        if (can_go(there)) {
            h_pos[h] = there;
            dfs(turn + 1, sum + score[there]);
            h_pos[h] = here;
        }
    }
}
```
</details>

### 설명
문제 설명을 보면 '**말이 파란색 칸에서 이동을 시작하면 파란색 화살표를 타야 하고**' 라고 적혀있다.

파란색 칸에서는 빨간색 선을 타면 안된다.

문제를 잘 보자!!!

말이 4개이고, 총 10개의 주사위 수가 주어지므로 가능한 경우의 수는 4^10 으로 모든 경우를 시간 안에 다 세볼 수 있다.

***

## 20061. 모노미노도미노 2
> https://www.acmicpc.net/problem/20061

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>

int N, t, y, x, score;
bool green[6][4], blue[6][4];

void move(bool (*board)[4]);
int rem_block(bool (*board)[4]);
void clr_block(bool (*board)[4]);

void print(bool (*board)[4]) {
  printf("\n");
  for (int i = 0; i < 6; ++i) {
    for (int j = 0; j < 4; ++j) {
      printf("%d", board[i][j]);
    }
    printf("\n");
  }
}

int main() {
  scanf("%d", &N);
  while (N--) {
    scanf("%d %d %d", &t, &y, &x);
    if (t == 1) {
      green[1][x] = true;
      blue[1][y] = true;
    }
    else if (t == 2) {
      green[1][x] = green[1][x + 1] = true;
      blue[0][y] = blue[1][y] = true;
    }
    else {
      green[0][x] = green[1][x] = true;
      blue[1][y] = blue[1][y + 1] = true;
    }

    move(green); move(blue);
    score += rem_block(green);
    score += rem_block(blue);
    clr_block(green);
    clr_block(blue);
  }

  int cnt = 0;
  for (int y = 2; y < 6; ++y) {
    for (int x = 0; x < 4; ++x) {
      if (green[y][x]) ++cnt;
      if (blue[y][x]) ++cnt;
    }
  }

  printf("%d\n%d", score, cnt);
  return 0;
}

void clr_block(bool (*board)[4]) {
  int cnt = 0;
  for (int y = 0; y < 2; ++y) {
    for (int x = 0; x < 4; ++x) {
      if (board[y][x]) {
        ++cnt;
        break;
      }
    }
  }
  if (cnt == 0) return;

  for (int y = 5; y >= 2; --y) {
    for (int x = 0; x < 4; ++x) {
      board[y][x] = board[y - cnt][x];
    }
  }

  for (int y = 0; y < 2; ++y) {
    for (int x = 0; x < 4; ++x) {
      board[y][x] = false;
    }
  }
}

int rem_block(bool (*board)[4]) {
  int ret = 0;

  while (true) {
    bool rem = false;

    for (int y = 5; y >= 2; --y) {
      bool fill = true;
      for (int x = 0; x < 4; ++x) {
        if (!board[y][x]) {
          fill = false;
          break;
        }
      }

      if (fill) {
        ++ret; rem = true;
        for (int yy = y; yy >= 1; --yy) {
          for (int x = 0; x < 4; ++x) {
            board[yy][x] = board[yy - 1][x];
          }
        }
        for (int x = 0; x < 4; ++x) {
          board[0][x] = false;
        }
        break;
      }
    }

    if (!rem) break;
  }

  return ret;
}

void move(bool (*board)[4]) {
  // 블록이 어디까지 이동하는지 확인
  int y;
  for (y = 2; y < 6; ++y) {
    bool check = true;
    for (int x = 0; x < 4; ++x) {
      if (board[1][x] && board[y][x]) {
        check = false;
        break;
      }
    }
    if (!check) {
      break;
    }
  }
  --y;

  // 블록 이동
  if (y >= 2) {
    for (int x = 0; x < 4; ++x) {
      board[y][x] |= board[1][x];
      board[1][x] = false;
    }
    for (int x = 0; x < 4; ++x) {
      board[y - 1][x] |= board[0][x];
      board[0][x] = false;
    }
  }
}
```
</details>

### 설명
1. 파란색 보드를 6행 4열의 보드로 뒤집어서 생각하면, 초록색 보드와 동일한 코드로 작업을 수행할 수 있다.

2. 이러한 복잡한 구현 문제는 절대 한번에 풀지 말고, 출력 함수를 먼저 만든 후, 매 기능을 구현할 때마다 출력을 하며 디버깅해야한다.
***

## 19236. 청소년 상어
> https://www.acmicpc.net/problem/19236

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

const int dy[8] = { -1, -1, 0, 1, 1, 1, 0, -1 }, dx[8] = { 0, -1, -1, -1, 0, 1, 1, 1 };

struct Pos {
  int y, x;
};

struct Fish {
  int y, x, d;
  bool eaten;
};

struct Info {
  int fishIdx[4][4]; // 0: 먹힘, 1 ~ 16: 물고기 번호
  Fish fishes[17];
  Fish shark;
};

int init(Info& info);

int ret = 0;
vector<Pos> getCands(Info& info);
void fishMove(Info& info);
void dfs(Info& info, int sum); // 현재 정보가 info로 주어지고, 지금까지의 물고기 번호 합이 sum일 때, 가능한 모든 경우의 수를 진행한 후 ret 갱신

int main() {
  Info info;
  int sum = init(info);
  dfs(info, sum);
  cout << ret;
  return 0;
}

void dfs(Info& info, int sum) {
  // 1. 물고기 이동
  fishMove(info);

  // 2. 상어 이동 && 물고기 먹기
  vector<Pos> cands = getCands(info);
  if (cands.size() == 0) {
    ret = max(ret, sum);
    return;
  }

  for (Pos& pos : cands) {
    int num = info.fishIdx[pos.y][pos.x];
    Fish& fish = info.fishes[num];
    Info nextInfo = info;
    nextInfo.shark = fish;
    nextInfo.fishIdx[pos.y][pos.x] = 0;
    nextInfo.fishes[num].eaten = true;
    dfs(nextInfo, sum + num);
  }
}

void fishMove(Info& info) {
  for (int num = 1; num <= 16; ++num) {
    Fish& fish = info.fishes[num];
    if (fish.eaten) continue;
    for (int i = 0; i < 8; ++i, fish.d = (fish.d + 1) % 8) {
      Pos there = { fish.y + dy[fish.d], fish.x + dx[fish.d] };
      if (there.y < 0 || there.y >= 4 || there.x < 0 || there.x >= 4) continue;
      if (info.shark.y == there.y && info.shark.x == there.x) continue;
      if (info.fishIdx[there.y][there.x] == 0) {
        info.fishIdx[there.y][there.x] = num;
        info.fishIdx[fish.y][fish.x] = 0;
        fish.y = there.y; fish.x = there.x;
        break;
      }
      else {
        int thereNum = info.fishIdx[there.y][there.x];
        swap(info.fishIdx[there.y][there.x], info.fishIdx[fish.y][fish.x]);
        swap(info.fishes[thereNum].y, info.fishes[num].y);
        swap(info.fishes[thereNum].x, info.fishes[num].x);
        break;
      }
    }
  }
}

vector<Pos> getCands(Info& info) {
  vector<Pos> ret;
  for (int i = 1; i <= 3; ++i) {
    int yy = info.shark.y + dy[info.shark.d] * i;
    int xx = info.shark.x + dx[info.shark.d] * i;
    if (yy < 0 || yy >= 4 || xx < 0 || xx >= 4) break;
    if (info.fishIdx[yy][xx] != 0) {
      ret.push_back({ yy, xx });
    }
  }
  return ret;
}

int init(Info& info) {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  int sum = 0;

  for (int y = 0; y < 4; ++y) {
    for (int x = 0; x < 4; ++x) {
      int a, b;
      cin >> a >> b;
      info.fishIdx[y][x] = a;
      info.fishes[a] = { y, x, b - 1, false };
    }
  }
  info.shark = info.fishes[info.fishIdx[0][0]];
  info.fishes[info.fishIdx[0][0]].eaten = true;
  sum += info.fishIdx[0][0];
  info.fishIdx[0][0] = 0;

  return sum;
}
```
</details>

### 설명
1. dfs로 가능한 모든 경우를 탐색하며 최댓값을 찾는 문제.

2. 적절한 print 함수를 만들어서, 한 기능이 완성되면 테스트하여 디버깅하며 구현해야 한다.

3. ~~dfs 함수 내부에서 구조체 원본을 백업해놓을때, 우선적으로는 스택에 할당하되, 구현이 끝나고 나서 힙에 할당하는 방식으로 바꾸면 스택 공간을 절약할 수 있다.~~

   ~~new / delete 연산자로 동적 할당 / 메모리 해제 를 할 수 있다.~~

4. dfs 탐색 최대 깊이 x Info 구조체 크기 만큼만 스택 공간이 할당되므로 힙에 할당까지 안해도 된다.

***

## 19237. 어른 상어
> https://www.acmicpc.net/problem/19237

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

struct Shark {
  int y, x, d; // 0: 위, 1: 아, 2:왼, 3: 오
  bool out;
};
int movedNum[20][20];
Shark sharks[401];

int N, M, k, sharkCount;
int prio[401][4][4];
void init();

struct Smell {
  int num, left;
};
Smell smells[20][20];

void moveShark(int num);

void print() {
  cout << "SMELL" << endl;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      cout << smells[y][x].left << ' ';
    }
    cout << endl;
  }
  cout << "SMELLNUM" << endl;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      cout << smells[y][x].num << ' ';
    }
    cout << endl;
  }
  cout << "NUM" << endl;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      cout << movedNum[y][x] << ' ';
    }
    cout << endl;
  }
}

int main() {
  init();

  for (int t = 1; t <= 1000; ++t) {
    // 1. 상어 이동 && 쫓아내기
    memset(movedNum, 0, sizeof(movedNum));
    for (int num = 1; num <= M; ++num) {
      if (sharks[num].out) continue;
      moveShark(num);
    }
    if (sharkCount == 1) {
      cout << t;
      return 0;
    }
    // 2. 이동한 곳에 냄새 뿌리기 (3번 고려해서 k+1 뿌리기)
    for (int num = 1; num <= M; ++num) {
      if (sharks[num].out) continue;
      smells[sharks[num].y][sharks[num].x].num = num;
      smells[sharks[num].y][sharks[num].x].left = k + 1;
    }
    // 3. 기존 냄새 left 줄이기
    for (int y = 0; y < N; ++y) {
      for (int x = 0; x < N; ++x) {
        if (smells[y][x].left == 0) continue;
        --smells[y][x].left;
      }
    }
  }

  cout << -1;
  return 0;
}

void moveShark(int num) {
  Shark& shark = sharks[num];

  // 냄새 없는 칸 탐색
  for (int p = 0; p < 4; ++p) {
    int d = prio[num][shark.d][p];
    int yy = shark.y + dy[d];
    int xx = shark.x + dx[d];
    if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
    // 냄새 없는 칸 발견
    if (smells[yy][xx].left == 0) {
      // 현재 비어있음
      if (movedNum[yy][xx] == 0) {
        movedNum[yy][xx] = num;
        shark.y = yy; shark.x = xx; shark.d = d;
      }
      // num보다 숫자 큰 상어 들어있음
      else if (movedNum[yy][xx] > num) {
        sharks[movedNum[yy][xx]].out = true;
        --sharkCount;
        movedNum[yy][xx] = num;
        shark.y = yy; shark.x = xx; shark.d = d;
      }
      // num보다 숫자 작은 상어 들어있음
      else {
        shark.out = true;
        --sharkCount;
      }
      return;
    }
  }

  // 냄새 없는 칸이 없음
  for (int p = 0; p < 4; ++p) {
    int d = prio[num][shark.d][p];
    int yy = shark.y + dy[d];
    int xx = shark.x + dx[d];
    if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
    // 번호 num 상어가 뿌린 냄새 칸
    if (smells[yy][xx].num == num) {
      movedNum[yy][xx] = num;
      shark.y = yy; shark.x = xx; shark.d = d;
      return;
    }
  }
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N >> M >> k;
  sharkCount = M;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      int num; cin >> num;
      if (num) {
        sharks[num] = { y, x, 0, false };
        smells[y][x] = { num, k };
      }
    }
  }
  for (int num = 1; num <= M; ++num) {
    cin >> sharks[num].d;
    --sharks[num].d;
  }

  for (int num = 1; num <= M; ++num) {
    for (int d = 0; d < 4; ++d) {
      for (int p = 0; p < 4; ++p) {
        cin >> prio[num][d][p];
        --prio[num][d][p];
      }
    }
  }
}
```
</details>

### 설명
복잡한 구현 문제.

구현 중간중간에 print() 함수를 쓰면서, 오류 없이 만들어가는게 중요하다.

***

## 19238. 스타트 택시
> https://www.acmicpc.net/problem/19238

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
#include <queue>
#include <algorithm>
using namespace std;

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };
struct Pos {
  int y, x;
  bool operator< (const Pos& rhs) const {
    if (y != rhs.y) return y < rhs.y;
    return x < rhs.x;
  }
};
struct Info {
  Pos from, to;
  bool arrived;
  bool operator< (const Info& rhs) const {
    return from < rhs.from;
  }
};
int N, M, fuel, t_y, t_x;
char board[20][20];
vector<Info> guests;
void input();

int dist[20][20];
void get_dist();

int get_min_guest();

void print_dist() {
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      printf("%3d", dist[y][x]);
    }
    printf("\n");
  }
}

int main() {
  input();

  for (int turn = 0; turn < M; ++turn) {
    // 1. 택시로부터 모든 칸까지의 거리 구하기
    get_dist();
    // 2. 가장 가까운 손님의 인덱스 구하기
    int g_idx = get_min_guest();
    if (g_idx == -1) {
      printf("-1");
      return 0;
    }
    // 3. 택시 손님까지 이동
    Info& g = guests[g_idx];
    fuel -= dist[g.from.y][g.from.x];
    if (fuel < 0) {
      printf("-1");
      return 0;
    }
    t_y = g.from.y; t_x = g.from.x;
    // 4. 손님 위치부터 목적지까지의 거리 계산
    get_dist();
    // 5. 택시 목적지까지 이동
    fuel -= dist[g.to.y][g.to.x];
    if (fuel < 0) {
      printf("-1");
      return 0;
    }
    fuel += dist[g.to.y][g.to.x] * 2;
    t_y = g.to.y; t_x = g.to.x;

    // 6. 손님 도착 체크
    g.arrived = true;
  }
  printf("%d", fuel);
  return 0;
}

int get_min_guest() {
  int min_dist = 987654321;
  for (Info& g : guests) {
    if (g.arrived) continue;
    if (dist[g.from.y][g.from.x] == -1) {
      return -1;
    }
    min_dist = min(min_dist, dist[g.from.y][g.from.x]);
  }

  for (int i = 0; i < guests.size(); ++i) {
    Info& g = guests[i];
    if (g.arrived) continue;
    if (dist[g.from.y][g.from.x] == min_dist) {
      return i;
    }
  }

  return -1;
}

void get_dist() {
  memset(dist, -1, sizeof(dist));
  queue<Pos> q;

  dist[t_y][t_x] = 0;
  q.push({ t_y, t_x });

  while (!q.empty()) {
    Pos here = q.front(); q.pop();
    for (int d = 0; d < 4; ++d) {
      int yy = here.y + dy[d], xx = here.x + dx[d];
      if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
      if (dist[yy][xx] != -1 || board[yy][xx] == 1) continue;
      dist[yy][xx] = dist[here.y][here.x] + 1;
      q.push({ yy, xx });
    }
  }
}

void input() {
  scanf("%d %d %d", &N, &M, &fuel);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      scanf("%hhd", &board[i][j]);
    }
  }
  scanf("%d %d", &t_y, &t_x);
  --t_y; --t_x;

  guests.resize(M);
  for (int i = 0; i < M; ++i) {
    scanf("%d %d %d %d", &guests[i].from.y, &guests[i].from.x, &guests[i].to.y, &guests[i].to.x);
    --guests[i].from.y; --guests[i].from.x; --guests[i].to.y; --guests[i].to.x;
    guests[i].arrived = false;
  }
  sort(guests.begin(), guests.end());
}

#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>
#include <cstring>
using namespace std;

struct Pos {
  int y, x;
  bool operator==(const Pos& rhs) const {
    return y == rhs.y && x == rhs.x;
  }
  bool operator<(const Pos& rhs) const {
    return (y != rhs.y ? y < rhs.y : x < rhs.x);
  }
};

struct Customer {
  Pos to;
  bool arrived;
};

const int dy[4] = { -1, 1, 0 , 0 }, dx[4] = { 0, 0, -1, 1 };

int N, M, F;
bool board[20][20];
Pos taxi;
int customerIdx[20][20];
vector<Customer> customers;
void init();

int getDist(Pos from, Pos to);
bool taxiMove();

int main() {
  init();

  for (int i = 0; i < M; ++i) {
    if (!taxiMove()) {
      cout << -1;
      return 0;
    }
  }

  cout << F;

  return 0;
}

bool taxiMove() {
  queue<Pos> q;
  vector<vector<int>> dist(N, vector<int>(N, -1));
  vector<Pos> cands;

  q.push(taxi);
  dist[taxi.y][taxi.x] = 0;

  while (!q.empty()) {
    Pos here = q.front(); q.pop();

    int cIdx = customerIdx[here.y][here.x];
    if (cIdx != -1 && !customers[cIdx].arrived) {
      if (cands.empty() || dist[cands[0].y][cands[0].x] == dist[here.y][here.x]) {
        cands.push_back(here);
      }
      else {
        break;
      }
    }

    for (int d = 0; d < 4; ++d) {
      Pos there = { here.y + dy[d], here.x + dx[d] };
      if (there.y < 0 || there.y >= N || there.x < 0 || there.x >= N) continue;
      if (board[there.y][there.x] || dist[there.y][there.x] != -1) continue;
      q.push(there);
      dist[there.y][there.x] = dist[here.y][here.x] + 1;
    }
  }

  if (cands.empty()) return false;

  sort(cands.begin(), cands.end());

  int cIdx = customerIdx[cands[0].y][cands[0].x];
  Customer& customer = customers[cIdx];

  int toDest = getDist(cands[0], customer.to);
  if (toDest == -1) return false;
  int totalDist = dist[cands[0].y][cands[0].x] + toDest;

  if (F >= totalDist) {
    F -= totalDist;
    F += 2 * toDest;
    customer.arrived = true;
    taxi = customer.to;
    return true;
  }
  else {
    return false;
  }
}

int getDist(Pos from, Pos to) {
  queue<Pos> q;
  vector<vector<int>> dist(N, vector<int>(N, -1));
  q.push(from);
  dist[from.y][from.x] = 0;

  while (!q.empty()) {
    Pos here = q.front(); q.pop();
    for (int d = 0; d < 4; ++d) {
      Pos there = { here.y + dy[d], here.x + dx[d] };
      if (there.y < 0 || there.y >= N || there.x < 0 || there.x >= N) continue;
      if (board[there.y][there.x] || dist[there.y][there.x] != -1) continue;
      if (there == to) {
        return dist[here.y][here.x] + 1;
      }
      dist[there.y][there.x] = dist[here.y][here.x] + 1;
      q.push(there);
    }
  }

  return -1;
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  memset(customerIdx, -1, sizeof(customerIdx));

  cin >> N >> M >> F;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      cin >> board[i][j];
    }
  }
  cin >> taxi.y >> taxi.x;
  --taxi.y; --taxi.x;

  customers = vector<Customer>(M);
  for (int c = 0; c < M; ++c) {
    int fromY, fromX, toY, toX;
    cin >> fromY >> fromX >> toY >> toX;
    --fromY; --fromX; --toY; --toX;
    customers[c] = { {toY, toX}, false };
    customerIdx[fromY][fromX] = c;
  }
}
```
</details>

### 설명
구현 + bfs 문제.

1. 문제에서 필요한 정보 (좌표, 두 좌표를 갖는 손님) 를 구조체로 표현하여 코드를 작성하면 깔끔하다.

2. **문제를 푸는 도중도중에 반드시 print 해가며 올바르게 진행중인지 확인해야한다.**

3. 2차원 좌표에서 벽이 있는 경우 막혀서 못가는 경우를 항상 고려해야한다 !!

4. 좌표를 입력받고 미리 y가 커지는 순 -> x가 커지는 순으로 정렬 후 문제를 푸는 방법도 있다.
***

## 20055. 컨베이어 벨트 위의 로봇
> https://www.acmicpc.net/problem/20055

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
using namespace std;

int N, K, A[200];
void init();

int zeroCnt = 0;
bool robots[100];

int main() {
	init();

	for (int turn = 1; ; ++turn) {
		// 1 - 1. 벨트 회전
		int tmp = A[2 * N - 1];
		for (int i = 2 * N - 2; i >= 0; --i) {
			A[i + 1] = A[i];
		}
		A[0] = tmp;

		// 1 - 2. 로봇 회전
		for (int i = N - 1; i >= 1; --i) {
			robots[i] = robots[i - 1];
		}
		robots[0] = robots[N - 1] = false;

		// 2. 로봇 이동
		for (int i = N - 2; i >= 1; --i) {
			if (robots[i] && !robots[i + 1] && A[i + 1]) {
				robots[i] = false;
				robots[i + 1] = true;
				--A[i + 1];
				if (A[i + 1] == 0) ++zeroCnt;
			}
		}
		robots[N - 1] = false;

		// 3. 로봇 올리기
		if (A[0]) {
			robots[0] = true;
			--A[0];
			if (A[0] == 0) ++zeroCnt;
		}

		// 4. 내구도 0인 칸 K개 이상인지 확인
		if (zeroCnt >= K) {
			cout << turn;
			return 0;
		}
	}

	return 0;
}

void init() {
	ios_base::sync_with_stdio(false);
	cin.tie(NULL);

	cin >> N >> K;
	for (int i = 0; i < 2 * N; ++i) {
		cin >> A[i];
	}
}
```
</details>

### 설명
단순한 구현 문제.

***

## 20056. 마법사 상어와 파이어볼
> https://www.acmicpc.net/problem/20056

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

const int dy[8] = { -1, -1, 0, 1, 1, 1, 0, -1 }, dx[8] = { 0, 1, 1, 1, 0, -1, -1, -1 };

struct Ball {
	int y, x, m, s, d;
};

struct Info {
	int m_sum, s_sum, cnt;
	int d;
	bool same;
};

int N, M, K;
vector<Ball> balls;
void init();

vector<vector<Info>> merged;
void moveBall(Ball& ball);

int main() {
	init();

	for (int order = 0; order < K; ++order) {
		// 1. 파이어볼 이동
		merged = vector<vector<Info>>(N, vector<Info>(N, { 0, 0, 0, -1, true }));
		for (Ball& ball : balls) {
			moveBall(ball);
		}
		// 2. 2개 이상의 파이어볼 합치기
		vector<Ball> newBalls;
		for (int y = 0; y < N; ++y) {
			for (int x = 0; x < N; ++x) {
				Info& info = merged[y][x];
				if (info.cnt == 0) continue;
				else if (info.cnt == 1) {
					newBalls.push_back({ y, x, info.m_sum, info.s_sum, info.d });
				}
				else if (info.m_sum >= 5) {
					const int newDir[2][4] = { {0, 2, 4, 6}, {1, 3, 5, 7} };
					for (int i = 0; i < 4; ++i) {
						newBalls.push_back({ y, x, info.m_sum / 5, info.s_sum / info.cnt, newDir[(info.same ? 0 : 1)][i] });
					}
				}
			}
		}
		balls = newBalls;
	}

	int ret = 0;
	for (Ball& ball : balls) {
		ret += ball.m;
	}
	cout << ret;

	return 0;
}

void moveBall(Ball& ball) {
	int yy = ball.y + dy[ball.d] * (ball.s % N);
	int xx = ball.x + dx[ball.d] * (ball.s % N);
	if (yy < 0) yy += N;
	else if (yy >= N) yy -= N;
	if (xx < 0) xx += N;
	else if (xx >= N) xx -= N;

	merged[yy][xx].m_sum += ball.m;
	merged[yy][xx].s_sum += ball.s;
	++merged[yy][xx].cnt;

	if (merged[yy][xx].d != -1 && merged[yy][xx].d % 2 != ball.d % 2) {
		merged[yy][xx].same = false;
	}
	merged[yy][xx].d = ball.d;
}

void init() {
	ios_base::sync_with_stdio(false);
	cin.tie(NULL);

	cin >> N >> M >> K;
	for (int i = 0; i < M; ++i) {
		int y, x, m, s, d;
		cin >> y >> x >> m >> s >> d;
		--y; --x;
		balls.push_back({ y, x, m, s, d });
	}
}
```
</details>

### 설명
MOD 연산을 사용할 때에는 항상 주의해야 한다.

**MOD 연산의 대상 값이 다른 연산에 사용되는지 꼭 확인해야 한다.**

이 문제에서는 (파이어볼의 속력의 합) / (파이어볼의 개수) 를 새로운 속력으로 사용하므로, 파이어볼의 속력을 무턱대고 MOD N 값으로 저장하면 틀리게 된다.

***

## 20057. 마법사 상어와 토네이도
> https://www.acmicpc.net/problem/20057

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
using namespace std;

struct Pos {
  int y, x;
};

const int dy[4] = { 0, 1, 0, -1 }, dx[4] = { -1, 0, 1, 0 };

int N, A[500][500];
void init();

int sand = 0;
void tornado(Pos pos, int d, int len);
void blow(Pos from, int d);

int main() {
  init();

  int y = N / 2, x = N / 2;
  for (int len = 1; len < N; len += 2) {
    tornado({ y, x }, 0, len);
    x -= len;
    tornado({ y, x }, 1, len);
    y += len;
    tornado({ y, x }, 2, len + 1);
    x += len + 1;
    tornado({ y, x }, 3, len + 1);
    y -= len + 1;
  }
  tornado({ y, x }, 0, N - 1);

  cout << sand;

  return 0;
}

const int delta[9][3] = { // {left, center, right}
  {1, 0, 0}, {1, 1, 0}, {1, 2, 0}, {2, 1, 0}, {0, 3, 0}, {0, 0, 1}, {0, 1, 1}, {0, 2, 1}, {0, 1, 2}
};
const int per[9] = { 1, 7, 10, 2, 5, 1, 7, 10, 2 };

void blow(Pos from, int d) {
  int dl = (d + 1) % 4, dr = (d + 3) % 4;
  int sum = 0;
  Pos to = { from.y + dy[d], from.x + dx[d] };

  for (int i = 0; i < 9; ++i) {
    Pos there;
    there.y = { from.y + dy[dl] * delta[i][0] + dy[d] * delta[i][1] + dy[dr] * delta[i][2] };
    there.x = { from.x + dx[dl] * delta[i][0] + dx[d] * delta[i][1] + dx[dr] * delta[i][2] };
    int s = A[to.y][to.x] * per[i] / 100;
    if (there.y < 0 || there.y >= N || there.x < 0 || there.x >= N) {
      sand += s;
    }
    else {
      A[there.y][there.x] += s;
    }
    sum += s;
  }
  Pos a = { from.y + dy[d] * 2, from.x + dx[d] * 2 };
  if (a.y < 0 || a.y >= N || a.x < 0 || a.x >= N) {
    sand += A[to.y][to.x] - sum;
  }
  else {
    A[a.y][a.x] += A[to.y][to.x] - sum;
  }

  A[to.y][to.x] = 0;
}

void tornado(Pos pos, int d, int len) {
  for (int i = 0; i < len; ++i) {
    blow(pos, d);
    pos.y += dy[d]; pos.x += dx[d];
  }
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N;
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      cin >> A[i][j];
    }
  }
}
```
</details>

### 설명
2차원 배열 위에서 특정 방향으로의 점 이동을 연습할 수 있는 문제

d 와 dy[] dx[] 의 관계를 잘 생각하자.

d의 값에는 방향이 정해지고, 그 방향에 맞게 y변화량 및 x변화량을 y[d] 와 dx[d] 에 넣어주는 것이다.

***

## 20058. 마법사 상어와 파이어스톰
> https://www.acmicpc.net/problem/20058

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <cstring>
#include <queue>
#include <algorithm>
using namespace std;

struct Pos { int y, x; };

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

int N, Q, L, A[64][64], tmp[64][64];
void init();

void rotate(int y, int x, int l);
void checkIce(int y, int x);

bool discovered[64][64];
int getNum(int y, int x);

int main() {
  init();
  while (Q--) {
    cin >> L;

    // 1. 부분 격자 시계 방향 90도 회전
    for (int y = 0; y < (1 << N); y += (1 << L)) {
      for (int x = 0; x < (1 << N); x += (1 << L)) {
        rotate(y, x, (1 << L));
      }
    }

    // 2. 인접한 얼음이 있는 칸 2개 이하인 얼음 1개 줄이기.
    for (int y = 0; y < (1 << N); ++y) {
      for (int x = 0; x < (1 << N); ++x) {
        checkIce(y, x);
      }
    }
    memcpy(A, tmp, sizeof(A));
  }

  // A[r][c]의 합 구하기
  int sum = 0;
  for (int i = 0; i < (1 << N); ++i) {
    for (int j = 0; j < (1 << N); ++j) {
      sum += A[i][j];
    }
  }
  cout << sum << '\n';

  int biggest = 1;
  // 가장 큰 덩어리가 차지하는 칸의 개수 출력
  for (int y = 0; y < (1 << N); ++y) {
    for (int x = 0; x < (1 << N); ++x) {
      if (discovered[y][x] || A[y][x] == 0) continue;
      biggest = max(biggest, getNum(y, x));
    }
  }
  cout << (biggest > 1 ? biggest : 0);
}

int getNum(int y, int x) {
  queue<Pos> q;
  q.push({ y, x });
  discovered[y][x] = true;
  int ret = 0;

  while (!q.empty()) {
    Pos here = q.front(); q.pop();
    ++ret;
    for (int d = 0; d < 4; ++d) {
      Pos there = { here.y + dy[d], here.x + dx[d] };
      if (there.y < 0 || there.y >= (1 << N) || there.x < 0 || there.x >= (1 << N)) continue;
      if (discovered[there.y][there.x] || A[there.y][there.x] == 0) continue;
      discovered[there.y][there.x] = true;
      q.push(there);
    }
  }

  return ret;
}

void checkIce(int y, int x) {
  if (A[y][x] == 0) {
    tmp[y][x] = 0;
    return;
  }
  int cnt = 0;
  for (int d = 0; d < 4; ++d) {
    int yy = y + dy[d], xx = x + dx[d];
    if (yy < 0 || yy >= (1 << N) || xx < 0 || xx >= (1 << N)) continue;
    if (A[yy][xx] >= 1) ++cnt;
  }
  if (cnt <= 2) {
    tmp[y][x] = A[y][x] - 1;
  }
  else {
    tmp[y][x] = A[y][x];
  }
}

void rotate(int y, int x, int l) {
  for (int i = 0; i < l; ++i) {
    for (int j = 0; j < l; ++j) {
      tmp[y + j][x + l - 1 - i] = A[y + i][x + j];
    }
  }
  for (int i = 0; i < l; ++i) {
    for (int j = 0; j < l; ++j) {
      A[y + i][x + j] = tmp[y + i][x + j];
    }
  }
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);

  cin >> N >> Q;
  for (int i = 0; i < (1 << N); ++i) {
    for (int j = 0; j < (1 << N); ++j) {
      cin >> A[i][j];
    }
  }
}
```
</details>

### 설명
구현 + bfs 문제.
struct와 전역변수를 적절히 사용하면 간단하게 코드를 작성할 수 있다.

***

## 21608. 상어 초등학교
> https://www.acmicpc.net/problem/21608

### 코드
<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct Pos { int y, x; };
const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

int N, like[401][4], board[20][20];
vector<int> order;
void init();

void sit(int num);

int main() {
  init();

  for (int turn = 0; turn < N * N; ++turn) {
    sit(order[turn]);
  }

  int sum = 0;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      int num = board[y][x];
      int cnt = 0;
      for (int d = 0; d < 4; ++d) {
        int yy = y + dy[d], xx = x + dx[d];
        if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
        for (int i = 0; i < 4; ++i) {
          if (board[yy][xx] == like[num][i]) {
            ++cnt;
            break;
          }
        }
      }
      if (cnt == 1) sum += 1;
      else if (cnt == 2) sum += 10;
      else if (cnt == 3) sum += 100;
      else if (cnt == 4) sum += 1000;
    }
  }

  cout << sum;

  return 0;
}

void sit(int num) {
  // 1. 비어있는 칸 중에서 좋아하는 학생이 인접한 칸에 가장 많은 칸 개수 구하기
  int fav_cnt[20][20] = { 0, };
  int max_fav_cnt = 0;

  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      if (board[y][x]) continue;
      int cnt = 0;
      for (int d = 0; d < 4; ++d) {
        int yy = y + dy[d], xx = x + dx[d];
        if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
        if (board[yy][xx] == 0) continue;
        for (int i = 0; i < 4; ++i) {
          if (like[num][i] == board[yy][xx]) {
            ++cnt;
            break;
          }
        }
      }
      fav_cnt[y][x] = cnt;
      max_fav_cnt = max(max_fav_cnt, cnt);
    }
  }

  // 2. 인접 좋아하는 학생 수가 max_fav_cnt 인 칸들 중에서 인접 비어있는 칸 제일 큰 수 찾기
  int emp_cnt[20][20] = { 0, };
  int max_emp_cnt = 0;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      if (board[y][x] || fav_cnt[y][x] != max_fav_cnt) continue;
      int cnt = 0;
      for (int d = 0; d < 4; ++d) {
        int yy = y + dy[d], xx = x + dx[d];
        if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
        if (board[yy][xx] == 0) ++cnt;
      }
      emp_cnt[y][x] = cnt;
      max_emp_cnt = max(max_emp_cnt, cnt);
    }
  }

  // 3. fav_cnt[y][x] == max_fav_cnt && emp_cnt[y][x] == max_emp_cnt 인 y, x 찾아서 집어 넣고 반환
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      if (board[y][x] == 0 && fav_cnt[y][x] == max_fav_cnt && emp_cnt[y][x] == max_emp_cnt) {
        board[y][x] = num;
        return;
      }
    }
  }
}

void init() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);
  cin >> N;
  for (int i = 0; i < N * N; ++i) {
    int num; cin >> num;
    order.push_back(num);
    for (int j = 0; j < 4; ++j) {
      cin >> like[num][j];
    }
  }
}
```
</details>

### 설명

조건 A B C 가 있다.

A를 만족하고, 그것이 여러개일 때 B를 만족하고, 그것이 여러개일 때 C를 만족하는 것을 찾아야 한다.

1. A를 만족하는 값을 찾고, 

2. A를 만족하는 값과 같은 위치 중에서 B를 만족하는 값을 찾고,

3. C를 만족하는 순서대로 A를 만족하는 값과 B를 만족하는 값을 갖는 위치를 발견하면 반환하면 된다.

***

## 21609. 상어 중학교
> https://www.acmicpc.net/problem/21609

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
#include <vector>
#include <algorithm>
using namespace std;

struct Pos {
  int y, x;
};
const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

int N, M, board[20][20], tmp[20][20], score;
void input();

vector<vector<Pos>> groups;

bool discovered1[20][20]; // 전체 bfs 에서의 방문 여부 체크
bool discovered2[20][20]; // 이번 bfs 에서의 방문 여부 체크
void find_group();
void bfs(int y, int x);

void erase_group();
void gravity();
void rotate();

void print(const char* str) {
  printf("\n%s\n", str);
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      printf("%3d", board[y][x]);
    }
    printf("\n");
  }
}

int main() {
  input();

  while (true) {
    // 1. 블록 그룹 찾기
    groups.clear();
    find_group();
    // 2. 그룹이 없으면 종료
    if (groups.empty()) break;
    // 3. 조건에 맞는 그룹 제거
    erase_group();
    // 4. 중력 작용
    gravity();
    // 5. 격자 반시계 회전
    rotate();
    // 6. 중력 작용
    gravity();
  }
  printf("%d", score);
  return 0;
}

void rotate() {
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      tmp[N - 1 - x][y] = board[y][x];
    }
  }
  memcpy(board, tmp, sizeof(board));
}

void gravity() {
  for (int x = 0; x < N; ++x) {
    int btm = N - 1;
    while (btm >= 0 && board[btm][x] != -2) --btm;
    for (int y = btm - 1; y >= 0; --y) {
      if (board[y][x] == -2) continue;
      if (board[y][x] >= 0) {
        board[btm--][x] = board[y][x];
        board[y][x] = -2;
      }
      else {
        btm = y - 1;
        while (btm >= 0 && board[btm][x] != -2) --btm;
        y = btm;
      }
    }
  }
}

void erase_group() {
  int max_group_size = 0;
  for (auto& v : groups) {
    max_group_size = max<int>(max_group_size, v.size());
  }

  vector<int> rbw_cnt(groups.size(), 0);
  int max_rbw_cnt = 0;
  for (int i = 0; i < groups.size(); ++i) {
    auto& v = groups[i];
    if (v.size() != max_group_size) continue; // 그룹의 크기가 max_group_size 인 그룹들로만 max_rbw_cnt 를 찾아야한다!!!!
    for (auto& p : v) {
      if (board[p.y][p.x] == 0) {
        ++rbw_cnt[i];
      }
    }
    max_rbw_cnt = max(max_rbw_cnt, rbw_cnt[i]);
  }

  for (int i = groups.size() - 1; i >= 0; --i) {
    if (groups[i].size() == max_group_size && rbw_cnt[i] == max_rbw_cnt) {
      for (auto& p : groups[i]) {
        board[p.y][p.x] = -2;
      }
      score += groups[i].size() * groups[i].size();
      return;
    }
  }
}

void find_group() {
  memset(discovered1, false, sizeof(discovered1));
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      if (!discovered1[y][x] && board[y][x] >= 1) {
        memset(discovered2, false, sizeof(discovered2));
        bfs(y, x);
      }
    }
  }
}

void bfs(int y, int x) {
  queue<Pos> q;
  vector<Pos> v;
  int color = board[y][x];

  discovered1[y][x] = discovered2[y][x] = true;
  q.push({ y, x });

  while (!q.empty()) {
    Pos here = q.front(); q.pop();
    v.push_back(here);
    for (int d = 0; d < 4; ++d) {
      int yy = here.y + dy[d], xx = here.x + dx[d];
      if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
      if (board[yy][xx] == -1 || board[yy][xx] == -2) continue;
      if (board[yy][xx] >= 1 && (discovered1[yy][xx] || board[yy][xx] != color)) continue;
      if (board[yy][xx] == 0 && discovered2[yy][xx]) continue;
      discovered1[yy][xx] = discovered2[yy][xx] = true;
      q.push({ yy, xx });
    }
  }

  if (v.size() >= 2) groups.push_back(v);
}

void input() {
  scanf("%d %d", &N, &M);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      scanf("%d", &board[i][j]);
    }
  }
}
```
</details>

### 설명
위 문제에서는 크기가 가장 큰 그룹을, 그러한 그룹이 여러개라면 무지개 블록이 제일 많은 그룹을, 그러한 그룹이 여러개라면 기준 블록이 가장 우측 하단에 있는 그룹을 찾아야 한다.

간단하게 풀기 위해서 그룹의 최대 크기를 구한 후, 최대 크기를 갖는 그룹들 중에서 최대 무지개 블록 수를 구한 후, 기준 블록이 우측 하단에 있는 그룹부터 차례로 순회하며 그룹의 크기가 앞에서 구한 최대 크기이고, 무지개 블록의 개수가 앞에서 구한 최대 무지개 블록 개수인 그룹을 찾으면 된다.

이때 최대 무지개 블록 수는 **최대 그룹 크기** 인 그룹들 중에서 구해야 한다는 것을 주의해야 한다.

***

## 21610. 마법사 상어와 비바라기
> https://www.acmicpc.net/problem/21610

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>

const int dy[8] = { 0, -1, -1, -1, 0, 1, 1, 1 }, dx[8] = { -1, -1, 0, 1, 1, 1, 0, -1 };
struct Pos {
  int y, x;
  void move(int d, int s);
};

int N, M, d, s, A[50][50];
bool moved[50][50];
Pos clouds[2500]; int c_num;
void input();

void print(const char* str) {
  printf("\n%s\n", str);
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      printf("%3d", A[y][x]);
    }
    printf("\n");
  }
}

int main() {
  input();

  for (int turn = 0; turn < M; ++turn) {
    scanf("%d %d", &d, &s);
    --d; s %= N;
    // 1. 구름 이동 && 2. 이동된 위치 표시 && 3. 물의 양 1 증가시키기
    memset(moved, false, sizeof(moved));
    for (int i = 0; i < c_num; ++i) {
      Pos& c = clouds[i];
      c.move(d, s);
      ++A[c.y][c.x];
      moved[c.y][c.x] = true;
    }
    // 4. 물복사버그 마법 시전
    for (int i = 0; i < c_num; ++i) {
      Pos& c = clouds[i];
      int cnt = 0;
      for (int d = 1; d <= 7; d += 2) {
        int yy = c.y + dy[d], xx = c.x + dx[d];
        if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
        if (A[yy][xx] > 0) ++cnt;
      }
      A[c.y][c.x] += cnt;
    }
    // 5. 구름 사라짐
    c_num = 0;
    // 6. 구름 생김
    for (int y = 0; y < N; ++y) {
      for (int x = 0; x < N; ++x) {
        if (moved[y][x]) continue;
        if (A[y][x] >= 2) {
          A[y][x] -= 2;
          clouds[c_num++] = { y, x };
        }
      }
    }
  }

  long long sum = 0;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      sum += A[y][x];
    }
  }

  printf("%lld", sum);
  return 0;
}

void Pos::move(int d, int s) {
  y += dy[d] * s; x += dx[d] * s;
  if (y < 0) y += N; if (y >= N) y -= N;
  if (x < 0) x += N; if (x >= N) x -= N;
}

void input() {
  scanf("%d %d", &N, &M);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      scanf("%d", &A[i][j]);
    }
  }
  clouds[c_num++] = { N - 1, 0 };
  clouds[c_num++] = { N - 1, 1 };
  clouds[c_num++] = { N - 2, 0 };
  clouds[c_num++] = { N - 2, 1 };
}
```
</details>

### 설명
간단한 구현 문제

최댓값이 정해져 있으므로, 최댓값만큼 전역 배열로 할당해 놓으면 빠르다.

***

## 21611. 마법사 상어와 블리자드
> https://www.acmicpc.net/problem/21611

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>

void print(const char* str);

const int dy[4] = { 0, 1, 0, -1 }, dx[4] = { -1, 0, 1, 0 };
int N, M, board[50][50], d, s;
int* ptr[2500];
int tmp[2500], exp_cnt[3];
void input();

void move();
bool explode();
void change();

int main() {
  input();
  for (int turn = 0; turn < M; ++turn) {
    scanf("%d %d", &d, &s);
    if (d == 1) d = 3;
    else if (d == 2) d = 1;
    else if (d == 3) d = 0;
    else d = 2;
    // 1. 구슬 파괴
    int y = N / 2, x = N / 2;
    while (s--) {
      y += dy[d]; x += dx[d];
      board[y][x] = 0;
    }
    // 2. 구슬 이동
    move();
    // 3. 구슬 폭발
    while (true) {
      if (!explode()) {
        break;
      }
      move();
    }
    // 4. 구슬 변경
    change();
  }

  int ret = exp_cnt[0] + exp_cnt[1] * 2 + exp_cnt[2] * 3;
  printf("%d", ret);

  return 0;
}

void change() {
  memset(tmp, 0, sizeof(tmp));
  int idx = 1;
  for (int i = 1; i < N * N; ) {
    if (*ptr[i] == 0) break;
    int j;
    for (j = i; j + 1 < N * N && *ptr[i] == *ptr[j + 1]; ++j);
    tmp[idx++] = j - i + 1;
    if (idx == N * N) break;
    tmp[idx++] = *ptr[i];
    if (idx == N * N) break;
    i = j + 1;
  }
  for (int i = 1; i < N * N; ++i) {
    *ptr[i] = tmp[i];
  }
}

bool explode() {
  bool check = false;
  for (int i = 1; i < N * N; ) {
    if (*ptr[i] == 0) break;
    int j;
    for (j = i; j + 1 < N * N && *ptr[i] == *ptr[j + 1]; ++j);
    if (j - i + 1 >= 4) {
      check = true;
      if (*ptr[i] <= 3) {
        exp_cnt[*ptr[i] - 1] += j - i + 1;
      }
      for (int k = i; k <= j; ++k) {
        *ptr[k] = 0;
      }
    }
    i = j + 1;
  }
  return check;
}

void move() {
  int btm = 1;
  while (btm < N * N && *ptr[btm] != 0) ++btm;
  for (int i = btm + 1; i < N * N; ++i) {
    if (*ptr[i] == 0) continue;
    *ptr[btm++] = *ptr[i];
    *ptr[i] = 0;
  }
}

void input() {
  scanf("%d %d", &N, &M);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      scanf("%d", &board[i][j]);
    }
  }
  int y = N / 2, x = N / 2, d = 0, idx = 0;
  ptr[idx++] = &board[y][x];

  for (int len = 1; ; ++len) {
    for (int i = 0; i < len; ++i) {
      y += dy[d]; x += dx[d];
      ptr[idx++] = &board[y][x];
    }
    d = (d + 1) % 4;

    for (int i = 0; i < len; ++i) {
      y += dy[d]; x += dx[d];
      ptr[idx++] = &board[y][x];
    }
    d = (d + 1) % 4;

    if (len == N - 1) {
      for (int i = 0; i < len; ++i) {
        y += dy[d]; x += dx[d];
        ptr[idx++] = &board[y][x];
      }
      break;
    }
  }
}

void print(const char* str) {
  printf("\n%s\n", str);
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      printf("%3d", board[y][x]);
    }
    printf("\n");
  }
}
```
</details>

### 설명
복잡한 구현 문제.

다시 푼 문제임에도 중간중간 실수가 많았다.

void print(const char* str); 함수를 정의하여 중간중간 출력하며 디버깅을 꼭 해야한다.

***

## 주사위 굴리기 2
> https://www.acmicpc.net/problem/23288

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
using namespace std;

void print(const char* str);

struct Pos { int y, x; };
enum Face { up, down, front, rgt, back, lft };
const int dy[4] = { 0, 1, 0, -1 }, dx[4] = { 1, 0, -1, 0 };
int dice[6] = { 1, 6, 5, 3, 2, 4 };
Pos pos;
int N, M, K, d_d, score;
char board[20][20];
void input();

void move_dice();

bool discovered[20][20];
int get_score(int y, int x);

int main() {
  input();
  for (int turn = 0; turn < K; ++turn) {
    // 1. 주사위 굴러가기
    move_dice();
    // 2. 점수 획득
    score += board[pos.y][pos.x] * get_score(pos.y, pos.x);
    // 3. 방향 전환
    if (dice[down] > board[pos.y][pos.x]) {
      d_d = (d_d + 1) % 4;
    }
    else if (dice[down] < board[pos.y][pos.x]) {
      d_d = (d_d + 3) % 4;
    }
  }

  printf("%d", score);
  return 0;
}

int get_score(int y, int x) {
  memset(discovered, false, sizeof(discovered));
  queue<Pos> q;
  int ret = 0;

  discovered[y][x] = true;
  q.push({ y, x });

  while (!q.empty()) {
    Pos here = q.front(); q.pop(); ++ret;
    for (int d = 0; d < 4; ++d) {
      int yy = here.y + dy[d], xx = here.x + dx[d];
      if (yy < 0 || yy >= N || xx < 0 || xx >= M) continue;
      if (discovered[yy][xx] || board[yy][xx] != board[here.y][here.x]) continue;
      discovered[yy][xx] = true;
      q.push({ yy, xx });
    }
  }

  return ret;
}

void move_dice() {
  int yy = pos.y + dy[d_d], xx = pos.x + dx[d_d];
  if (yy < 0 || yy >= N || xx < 0 || xx >= M) {
    d_d = (d_d + 2) % 4;
    move_dice();
    return;
  }
  // 1. 주사위 좌표 이동
  pos = { yy, xx };
  // 2. 주사위 위 숫자 변경
  int tmp[6];
  memcpy(tmp, dice, sizeof(dice));
  if (d_d == 0) { // 오른쪽으로 구름
    tmp[down] = dice[rgt];
    tmp[rgt] = dice[up];
    tmp[up] = dice[lft];
    tmp[lft] = dice[down];
  }
  else if (d_d == 1) { // 아래로 구름
    tmp[up] = dice[back];
    tmp[front] = dice[up];
    tmp[down] = dice[front];
    tmp[back] = dice[down];
  }
  else if (d_d == 2) { // 왼쪽으로 구름
    tmp[down] = dice[lft];
    tmp[rgt] = dice[down];
    tmp[up] = dice[rgt];
    tmp[lft] = dice[up];
  }
  else { // 위로 구름
    tmp[up] = dice[front];
    tmp[front] = dice[down];
    tmp[down] = dice[back];
    tmp[back] = dice[up];
  }
  memcpy(dice, tmp, sizeof(dice));
}

void input() {
  scanf("%d %d %d", &N, &M, &K);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < M; ++j) {
      scanf("%hhd", &board[i][j]);
    }
  }
}

void print(const char* str) {
  printf("\n%s\n", str);
  printf("Up: %d, Down: %d, Front: %d, Right: %d, Back: %d, Left: %d\n", dice[up], dice[down], dice[front], dice[rgt], dice[back], dice[lft]);
  printf("y: %d, x: %d, score: %d\n", pos.y, pos.x, score);
}
```
</details>

### 설명
구현 + bfs 문제.

enum 을 사용하면 헷갈리지 않고 코드를 작성할 수 있다.

***

## 23289. 온풍기 안녕!
> https://www.acmicpc.net/problem/23289

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>

void print(const char* str);

struct Pos { int y, x; };
const int dy[4] = { 0, 1, 0, -1 }, dx[4] = { 1, 0, -1, 0 }; // 오, 아래, 왼, 위

int R, C, K, W, board[20][20], heat[20][20], tmp[20][20];
bool wall[20][20][20][20];
Pos check_list[400]; int check_num;
void input();
bool range(int y, int x);
void set_heater(int y, int x, int d);

void adjust();
bool inspect();

int main() {
  input();

  for (int cho = 1; cho <= 100; ++cho) {
    // 1. 온풍기에서 바람 나옴
    for (int y = 0; y < R; ++y) {
      for (int x = 0; x < C; ++x) {
        board[y][x] += heat[y][x];
      }
    }
    // 2. 온도 조절
    adjust();
    // 3. 가장 바깥쪽 1 감소
    for (int y = 0; y < R; ++y) {
      if (board[y][0] >= 1) --board[y][0];
      if (board[y][C - 1] >= 1) --board[y][C - 1];
    }
    for (int x = 1; x < C - 1; ++x) {
      if (board[0][x] >= 1) --board[0][x];
      if (board[R - 1][x] >= 1) --board[R - 1][x];
    }
    // 4. 검사
    if (inspect()) {
      printf("%d", cho);
      return 0;
    }
  }
  printf("101");
  return 0;
}

bool inspect() {
  for (int i = 0; i < check_num; ++i) {
    Pos& p = check_list[i];
    if (board[p.y][p.x] < K) {
      return false;
    }
  }
  return true;
}

void adjust() {
  memset(tmp, 0, sizeof(tmp));
  for (int y = 0; y < R; ++y) {
    for (int x = 0; x < C; ++x) {
      tmp[y][x] += board[y][x];
      for (int d = 0; d < 4; ++d) {
        int yy = y + dy[d], xx = x + dx[d];
        if (!range(yy, xx) || wall[y][x][yy][xx] || board[yy][xx] >= board[y][x]) continue;
        int val = (board[y][x] - board[yy][xx]) / 4;
        tmp[y][x] -= val;
        tmp[yy][xx] += val;
      }
    }
  }
  memcpy(board, tmp, sizeof(board));
}

void input() {
  scanf("%d %d %d", &R, &C, &K);
  for (int y = 0; y < R; ++y) {
    for (int x = 0; x < C; ++x) {
      scanf("%d", &tmp[y][x]);
    }
  }
  scanf("%d", &W);
  for (int i = 0; i < W; ++i) {
    int y, x, t;
    scanf("%d %d %d", &y, &x, &t);
    --y; --x;
    if (t == 0) {
      wall[y][x][y - 1][x] = wall[y - 1][x][y][x] = true; // 주의!!
    }
    else {
      wall[y][x][y][x + 1] = wall[y][x + 1][y][x] = true;
    }
  }

  for (int y = 0; y < R; ++y) {
    for (int x = 0; x < C; ++x) {
      if (1 <= tmp[y][x] && tmp[y][x] <= 4) {
        if (tmp[y][x] == 1) set_heater(y, x, 0);
        else if (tmp[y][x] == 2) set_heater(y, x, 2);
        else if (tmp[y][x] == 3) set_heater(y, x, 3);
        else if (tmp[y][x] == 4) set_heater(y, x, 1);
      }
      else if (tmp[y][x] == 5) {
        check_list[check_num++] = { y, x };
      }
    }
  }
}

bool range(int y, int x) { return 0 <= y && y < R && 0 <= x && x < C; }

bool visited[20][20];
void set_heater(int y, int x, int ld, int d, int rd, int v) {
  if (v == 0 || visited[y][x]) return;
  visited[y][x] = true;

  int y1, x1, y2, x2;
  // 1. 온도 상승
  heat[y][x] += v;
  // 2. 좌측 대각 확인
  y1 = y + dy[ld]; x1 = x + dx[ld];
  y2 = y1 + dy[d]; x2 = x1 + dx[d];
  if (range(y1, x1) && range(y2, x2) && !wall[y][x][y1][x1] && !wall[y1][x1][y2][x2]) {
    set_heater(y2, x2, ld, d, rd, v - 1);
  }
  // 3. 정면 확인
  y1 = y + dy[d]; x1 = x + dx[d];
  if (range(y1, x1) && !wall[y][x][y1][x1]) {
    set_heater(y1, x1, ld, d, rd, v - 1);
  }
  // 4. 우측 대각 확인
  y1 = y + dy[rd]; x1 = x + dx[rd];
  y2 = y1 + dy[d]; x2 = x1 + dx[d];
  if (range(y1, x1) && range(y2, x2) && !wall[y][x][y1][x1] && !wall[y1][x1][y2][x2]) {
    set_heater(y2, x2, ld, d, rd, v - 1);
  }
}

void set_heater(int y, int x, int d) {
  memset(visited, false, sizeof(visited));
  int ld = (d + 3) % 4, rd = (d + 1) % 4;
  int yy = y + dy[d], xx = x + dx[d];
  if (range(yy, xx)) {
    set_heater(yy, xx, ld, d, rd, 5);
  }
}

void print(const char* str) {
  printf("\n%s\n", str);
  for (int y = 0; y < R; ++y) {
    for (int x = 0; x < C; ++x) {
      printf("%3d", board[y][x]);
    }
    printf("\n");
  }
}
```
</details>

### 설명
복잡한 구현 문제.

한 기능을 완성할 때마다 출력하며 디버깅해야한다.

2차원 배열에서의 방향과 이동의 관계를 연습할 수 있는 문제.

**wall[y1][x1][y2][x2] 가 true 면 wall[y2][x2][y1][x1] 도 참이어야 한다.**

   입력에서 한 방향만 주어지지만, 양쪽 방향 다 체크해놓아야 할 때 주의하자.

***

## 23290. 마법사 상어와 복제
> https://www.acmicpc.net/problem/23290

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>

const int dy[8] = { 0, -1, -1, -1, 0, 1, 1, 1 }, dx[8] = { -1, -1, 0, 1, 1, 1, 0, -1 };
const int s_dy[4] = { -1, 0, 1, 0 }, s_dx[4] = { 0, -1, 0, 1 };
int M, S, fish_num[4][4][8], tmp[4][4][8], copy_fish_num[4][4][8], smell[4][4], sy, sx;
void input();

void fish_move(int y, int x, int d);
void shark_move();

int main() {
  input();
  for (int turn = 0; turn < S; ++turn) {
    // 1. 물고기 복제
    memcpy(copy_fish_num, fish_num, sizeof(fish_num));

    // 2. 물고기 이동
    memset(tmp, 0, sizeof(tmp));
    for (int y = 0; y < 4; ++y) {
      for (int x = 0; x < 4; ++x) {
        for (int d = 0; d < 8; ++d) {
          if (fish_num[y][x][d] > 0) {
            fish_move(y, x, d);
          }
        }
      }
    }
    memcpy(fish_num, tmp, sizeof(fish_num));

    // 3. 상어 이동
    shark_move();

    // 4. 냄새 사라짐
    for (int y = 0; y < 4; ++y) {
      for (int x = 0; x < 4; ++x) {
        if (smell[y][x] > 0) {
          --smell[y][x];
        }
      }
    }

    // 5. 복제 마법 완료
    for (int y = 0; y < 4; ++y) {
      for (int x = 0; x < 4; ++x) {
        for (int d = 0; d < 8; ++d) {
          fish_num[y][x][d] += copy_fish_num[y][x][d];
        }
      }
    }
  }

  int sum = 0;
  for (int y = 0; y < 4; ++y) {
    for (int x = 0; x < 4; ++x) {
      for (int d = 0; d < 8; ++d) {
        sum += fish_num[y][x][d];
      }
    }
  }
  printf("%d", sum);

  return 0;
}

int visited_cnt[4][4], chosed[3], f_chosed[3], max_fish_num;
int get_fish_num(int y, int x) {
  int sum = 0;
  for (int d = 0; d < 8; ++d) {
    sum += fish_num[y][x][d];
  }
  return sum;
}
void dfs(int idx, int y, int x, int sum) {
  if (idx == 3) {
    if (max_fish_num < sum) {
      max_fish_num = sum;
      memcpy(f_chosed, chosed, sizeof(chosed));
    }
    return;
  }
  for (int d = 0; d < 4; ++d) {
    int yy = y + s_dy[d], xx = x + s_dx[d];
    if (yy < 0 || yy >= 4 || xx < 0 || xx >= 4) continue;
    chosed[idx] = d;
    ++visited_cnt[yy][xx];
    dfs(idx + 1, yy, xx, sum + (visited_cnt[yy][xx] == 1 ? get_fish_num(yy, xx) : 0));
    --visited_cnt[yy][xx];
  }
}
void shark_move() {
  memset(visited_cnt, 0, sizeof(visited_cnt));
  max_fish_num = -1;
  dfs(0, sy, sx, 0);

  for (int i = 0; i < 3; ++i) {
    sy += s_dy[f_chosed[i]];
    sx += s_dx[f_chosed[i]];
    for (int d = 0; d < 8; ++d) {
      if (fish_num[sy][sx][d] > 0) {
        fish_num[sy][sx][d] = 0;
        smell[sy][sx] = 3;
      }
    }
  }
}

void fish_move(int y, int x, int d) {
  int yy, xx, dd = d;
  for (int i = 0; i < 8; ++i) {
    yy = y + dy[dd], xx = x + dx[dd];
    if (0 <= yy && yy < 4 && 0 <= xx && xx < 4 && smell[yy][xx] == 0 && !(yy == sy && xx == sx)) {
      tmp[yy][xx][dd] += fish_num[y][x][d];
      return;
    }
    dd = (dd + 7) % 8;
  }
  tmp[y][x][d] += fish_num[y][x][d];
}

void input() {
  scanf("%d %d", &M, &S);
  for (int i = 0; i < M; ++i) {
    int y, x, d;
    scanf("%d %d %d", &y, &x, &d);
    ++fish_num[y - 1][x - 1][d - 1];
  }
  scanf("%d %d", &sy, &sx);
  --sy; --sx;
}
```
</details>

### 설명
문제를 코드로 옮길 때, 코드를 작성한 후 꼭 **문제에서 설명한 대로가 맞는지** 확인해야 한다.

이 문제에서는 물고기가 **상어의 위치** 로 이동하지 못한다는 조건이 있었는데, 빼먹었었다.

또한 문제에서 **상한값** 을 주어준다면, 왜 주었을지를 잘 생각해야 한다.

이 문제에서는 '격자 위에 있는 물고기의 수가 항상 1,000,000 이하인 입력만 주어진다.' 라고 하였는데,

이를 이용하면 물고기 수 배열을 int[4][4][8] 로 선언하면 (y, x) 에서의 dir 에 해당하는 물고기의 총 개수를 담을 수 있다. (1,000,000 이하임을 알고 있으므로 int에 담을 수 있음.)

dfs 에서 같은 정점을 여러번 방문할 수 있지만, 첫 번째 방문에서만 특정 행동을 수행하게 하려면 visited를 int 배열로 만든 후, 0일 때만 그 행동을 수행하게 하면 된다.

***

## 23291. 어항 정리
> https://www.acmicpc.net/problem/23291

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>

#define min(a, b) ((a) < (b) ? (a) : (b))
#define max(a, b) ((a) > (b) ? (a) : (b))

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

int N, K, arr[100][100], tmp[100][100], min_v, max_v;
void input();

void calc_min_max();
void calc1();
void calc2();
void calc3();
void calc4();

void print(const char* s);

int main() {
  input();
  for (int turn = 0; ; ++turn) {
    // 1. min_v, max_v 값 계산
    calc_min_max();
    // 2. 종료 조건
    if (max_v - min_v <= K) {
      printf("%d", turn);
      break;
    }
    // 3. 어항 정리
    for (int i = 0; i < N; ++i) {
      if (arr[0][i] == min_v) {
        ++arr[0][i];
      }
    }
    // 4. 어항 쌓기 1
    calc1();
    // 5. 물고기 수 조절
    calc2();
    // 6. 일렬로 놓기
    calc3();
    // 7. 어항 쌓기 2
    calc4();
    // 8. 물고기 수 조절
    calc2();
    // 9. 일렬로 놓기
    calc3();
  }
  return 0;
}

void calc4() {
  int l = 0, r = N / 2 + N / 4;
  for (int y = 1; y < 4; ++y) {
    if (y % 2) {
      for (int i = 0; i < N / 4; ++i) {
        arr[y][N - 1 - i] = arr[0][l + i];
        arr[0][l + i] = 0;
      }
    }
    else {
      for (int i = 0; i < N / 4; ++i) {
        arr[y][N / 2 + N / 4 + i] = arr[0][l + i];
        arr[0][l + i] = 0;
      }
    }
    l += N / 4;
  }
}

void calc3() {
  for (int i = 0; i < N; ++i) {
    tmp[0][i] = 0;
  }

  int idx = 0;
  for (int x = 0; x < N; ++x) {
    for (int y = 0; y < N; ++y) {
      if (arr[y][x] == 0) break;
      tmp[0][idx++] = arr[y][x];
      arr[y][x] = 0;
    }
  }

  for (int i = 0; i < N; ++i) {
    arr[0][i] = tmp[0][i];
  }
}

void calc2() {
  memcpy(tmp, arr, sizeof(arr));
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      if (arr[y][x] == 0) continue;
      for (int d = 0; d < 4; ++d) {
        int yy = y + dy[d], xx = x + dx[d];
        if (yy < 0 || yy >= N || xx < 0 || xx >= N || arr[yy][xx] == 0) continue;
        if (arr[y][x] > arr[yy][xx]) {
          int v = (arr[y][x] - arr[yy][xx]) / 5;
          tmp[y][x] -= v;
          tmp[yy][xx] += v;
        }
      }
    }
  }
  memcpy(arr, tmp, sizeof(arr));
}

void calc1() {
  int l = 0, y = 1, x = 1;
  while (true) {
    int r = l + x;
    if (r + y - 1 >= N) break;
    for (int i = 0; i < y; ++i) {
      for (int j = 0; j < x; ++j) {
        arr[1 + x - 1 - j][r + i] = arr[i][l + j];
        arr[i][l + j] = 0;
      }
    }
    if (y == x) ++y;
    else ++x;
    l = r;
  }
}

void calc_min_max() {
  min_v = max_v = arr[0][0];
  for (int i = 1; i < N; ++i) {
    min_v = min(min_v, arr[0][i]);
    max_v = max(max_v, arr[0][i]);
  }
}

void input() {
  scanf("%d %d", &N, &K);
  min_v = 987654321, max_v = 0;
  for (int i = 0; i < N; ++i) {
    scanf("%d", &arr[0][i]);
  }
}

void print(const char* str) {
  printf("\n%s\n", str);
  for (int i = N - 1; i >= 0; --i) {
    for (int j = 0; j < N; ++j) {
      printf("%3d", arr[i][j]);
    }
    printf("\n");
  }
}
```
</details>

### 설명
복잡한 구현 문제.

어떤 2차원 배열의 일부를 회전하여 다른 곳에 옮겨 붙일 때에는,

0부터 시작하는 인덱스를 회전 대상에 부여하여 회전했을때의 인덱스를 구한 후,

옮겨 붙이는 시작 인덱스에 더하여 주면 된다.

```cpp
for (int i = 0; i < y; ++i) {
  for (int j = 0; j < x; ++j) {
    arr[1 + x - 1 - j][r + i] = arr[i][l + j];
    arr[i][l + j] = 0;
  }
}
```

위 코드는, (0, l) 부터 y행 x 열의 도형을 시계방향으로 회전시켜 (1, r) 으로 이동시키는 코드이다.

따라서 (0 + i, l + j) -> (1 + x - 1 - j, i) 이 되게 된다.

***
