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
#include <cstdio>
#include <list>
using namespace std;

struct Tree {
  int age, cnt;
};
int N, M, K, A[10][10], food[10][10];
list<Tree> trees[10][10];
void input();

void calc1(int y, int x);
void calc2(int y, int x);

int main() {
  input();
  for (int year = 0; year < K; ++year) {
    // spring && summer
    for (int y = 0; y < N; ++y) {
      for (int x = 0; x < N; ++x) {
        calc1(y, x);
      }
    }
    // fall
    for (int y = 0; y < N; ++y) {
      for (int x = 0; x < N; ++x) {
        calc2(y, x);
      }
    }
    // winter
    for (int y = 0; y < N; ++y) {
      for (int x = 0; x < N; ++x) {
        food[y][x] += A[y][x];
      }
    }
  }

  int sum = 0;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      for (Tree& t : trees[y][x]) {
        sum += t.cnt;
      }
    }
  }
  printf("%d", sum);

  return 0;
}

void calc2(int y, int x) {
  list<Tree>& t = trees[y][x];
  for (Tree& tree : t) {
    if (tree.age % 5 != 0) continue;
    for (int dy = -1; dy <= 1; ++dy) {
      for (int dx = -1; dx <= 1; ++dx) {
        if (dy == 0 && dx == 0) continue;
        int yy = y + dy, xx = x + dx;
        if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
        if (!trees[yy][xx].empty() && trees[yy][xx].front().age == 1) {
          trees[yy][xx].front().cnt += tree.cnt;
        }
        else {
          trees[yy][xx].push_front({ 1, tree.cnt });
        }
      }
    }
  }
}

void calc1(int y, int x) {
  list<Tree>& t = trees[y][x];
  for (auto it = t.begin(); it != t.end(); ++it) {
    if (it->age * it->cnt <= food[y][x]) {
      food[y][x] -= it->age * it->cnt;
      ++it->age;
    }
    else {
      int eat_num = food[y][x] / it->age;
      int dead_num = it->cnt - eat_num;
      // 양분 먹기 && 죽은 나무 양분되기
      if (eat_num) {
        food[y][x] -= it->age * eat_num;
        food[y][x] += (it->age / 2) * dead_num;
        ++(it->age);
        it->cnt = eat_num;
        ++it;
      }
      while (it != t.end()) {
        food[y][x] += (it->age / 2) * it->cnt;
        it = t.erase(it);
      }
      break;
    }
  }
}

void input() {
  scanf("%d %d %d", &N, &M, &K);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      scanf("%d", &A[i][j]);
    }
  }
  while (M--) {
    int y, x, age;
    scanf("%d %d %d", &y, &x, &age);
    trees[y - 1][x - 1].push_front({ age, 1 });
  }
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      food[i][j] = 5;
    }
  }
}
```
</details>

### 설명
1. 리스트 활용 문제.
2. 자료구조 설계 문제.
   같은 위치에서 같은 나이의 나무들이 다수 생성될 가능성이 있으므로, {나무의 나이, 나무의 개수} 라는 자료구조를 만들면 시간 및 공간복잡도를 줄일 수 있다.
3. main 을 너무 단순하게 하려 하다보면 함수 크기가 너무 커져 헷갈려진다. 
   main 과 서브 루틴 함수의 사이즈를 적당히 조절하자.

***

## 16236. 아기 상어
> https://www.acmicpc.net/problem/16236

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
#include <algorithm>
using namespace std;

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };
struct Pos {
  int y, x;
};
Pos s_pos;
int s_size, s_cnt;

int N, board[20][20];
void input();

int dist[20][20], min_dist;
bool can_eat();
int eat_move();

int main() {
  input();
  int ret = 0;

  while (true) {
    if (!can_eat()) {
      break;
    }
    ret += eat_move();
  }

  printf("%d", ret);
  return 0;
}

int eat_move() {
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      if (1 <= board[y][x] && board[y][x] < s_size && dist[y][x] == min_dist) {
        s_pos = { y, x };
        board[y][x] = 0;
        --s_cnt;
        if (s_cnt == 0) {
          s_cnt = ++s_size;
        }
        return dist[y][x];
      }
    }
  }
  return -1;
}

bool can_eat() {
  memset(dist, -1, sizeof(dist));
  min_dist = 987654321;
  queue<Pos> q;

  q.push({ s_pos });
  dist[s_pos.y][s_pos.x] = 0;

  while (!q.empty()) {
    Pos here = q.front(); q.pop();
    if (1 <= board[here.y][here.x] && board[here.y][here.x] < s_size) {
      min_dist = min(min_dist, dist[here.y][here.x]);
    }
    for (int d = 0; d < 4; ++d) {
      int yy = here.y + dy[d], xx = here.x + dx[d];
      if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
      if (dist[yy][xx] != -1 || board[yy][xx] > s_size) continue;
      dist[yy][xx] = dist[here.y][here.x] + 1;
      q.push({ yy, xx });
    }
  }

  return min_dist != 987654321;
}

void input() {
  scanf("%d", &N);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      scanf("%d", &board[i][j]);
      if (board[i][j] == 9) {
        s_pos = { i, j };
        s_size = 2;
        s_cnt = 2;
        board[i][j] = 0;
      }
    }
  }
}
```
</details>

### 설명
bfs 구현 문제.

***

## 17144. 미세먼지 안녕!
> https://www.acmicpc.net/problem/17144

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };
int R, C, T, board[50][50], tmp[50][50];
int a_y;
void input();

void spread();
void aircondition();

int main() {
    input();

    for (int t = 0; t < T; ++t) {
        spread();
        aircondition();
    }

    int sum = 0;
    for (int y = 0; y < R; ++y) {
        for (int x = 0; x < C; ++x) {
            sum += board[y][x];
        }
    }
    printf("%d", sum);

    return 0;
}

void aircondition() {
    int y, x;
    // 1. 위쪽 방향
    y = a_y - 1, x = 0;
    while (y >= 1) {
        board[y][x] = board[y - 1][x];
        --y;
    }
    while (x <= C - 2) {
        board[y][x] = board[y][x + 1];
        ++x;
    }
    while (y <= a_y - 1) {
        board[y][x] = board[y + 1][x];
        ++y;
    }
    while (x >= 2) {
        board[y][x] = board[y][x - 1];
        --x;
    }
    board[a_y][x] = 0;

    // 2. 아래쪽 방향
    y = a_y + 2, x = 0;
    while (y <= R - 2) {
        board[y][x] = board[y + 1][x];
        ++y;
    }
    while (x <= C - 2) {
        board[y][x] = board[y][x + 1];
        ++x;
    }
    while (y >= a_y + 2) {
        board[y][x] = board[y - 1][x];
        --y;
    }
    while (x >= 2) {
        board[y][x] = board[y][x - 1];
        --x;
    }
    board[a_y + 1][x] = 0;
}

void spread() {
    memset(tmp, 0, sizeof(tmp));
    board[a_y][0] = board[a_y + 1][0] = -1;
    for (int y = 0; y < R; ++y) {
        for (int x = 0; x < C; ++x) {
            if (board[y][x] <= 0) continue;
            int s_cnt = 0;
            for (int d = 0; d < 4; ++d) {
                int yy = y + dy[d], xx = x + dx[d];
                if (yy < 0 || yy >= R || xx < 0 || xx >= C || board[yy][xx] == -1) continue;
                ++s_cnt;
                tmp[yy][xx] += board[y][x] / 5;
            }
            tmp[y][x] += board[y][x] - s_cnt * (board[y][x] / 5);
        }
    }
    memcpy(board, tmp, sizeof(board));
}

void input() {
    scanf("%d %d %d", &R, &C, &T);
    for (int y = 0; y < R; ++y) {
        for (int x = 0; x < C; ++x) {
            scanf("%d", &board[y][x]);
            if (board[y][x] == -1 && a_y == 0) {
                a_y = y;
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
#include <cstdio>
#include <cstring>
#include <vector>
using namespace std;

struct Info {
	int y, x, d, spd, size;
	bool eaten;
};
const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, 1, -1 };
int R, C, M, shark_idx[100][100];
vector<Info> sharks;

void input();

int catch_shark(int x);

void move_shark(Info& s, int dist);

void eat_shark();

int main() {
	input();
	int sum = 0;
	for (int x = 0; x < C; ++x) {
		// 1. 상어 잡기
		sum += catch_shark(x);
		// 2. 상어 이동
		for (int i = 0; i < sharks.size(); ++i) {
			move_shark(sharks[i], sharks[i].spd);
		}
		// 3. 상어끼리 잡아먹기
		eat_shark();
	}
	printf("%d", sum);
	return 0;
}

void eat_shark() {
	memset(shark_idx, -1, sizeof(shark_idx));
	for (int i = 0; i < sharks.size(); ++i) {
		Info& s = sharks[i];
		if (s.eaten) continue;
		if (shark_idx[s.y][s.x] == -1) {
			shark_idx[s.y][s.x] = i;
		}
		else {
			Info& s0 = sharks[shark_idx[s.y][s.x]];
			if (s0.size < s.size) {
				s0.eaten = true;
				shark_idx[s.y][s.x] = i;
			}
			else {
				s.eaten = true;
			}
		}
	}
}

void move_shark(Info& s, int dist) {
	if (s.d <= 1) {
		int yy = s.y + dy[s.d] * dist;
		if (0 <= yy && yy < R) {
			s.y = yy;
			return;
		}
		else if (yy < 0){
			int moved = s.y;
			s.y = 0;
			s.d = 1;
			move_shark(s, dist - moved);
		}
		else {
			int moved = R - 1 - s.y;
			s.y = R - 1;
			s.d = 0;
			move_shark(s, dist - moved);
		}
	}
	else {
		int xx = s.x + dx[s.d] * dist;
		if (0 <= xx && xx < C) {
			s.x = xx;
			return;
		}
		else if (xx < 0) {
			int moved = s.x;
			s.x = 0;
			s.d = 2;
			move_shark(s, dist - moved);
		}
		else {
			int moved = C - 1 - s.x;
			s.x = C - 1;
			s.d = 3;
			move_shark(s, dist - moved);
		}
	}
}

int catch_shark(int x) {
	int ret = 0;
	for(int y = 0; y < R; ++y) {
		if (shark_idx[y][x] != -1) {
			Info& shark = sharks[shark_idx[y][x]];
			if (shark.eaten) continue;
			shark_idx[y][x] = -1;
			ret = shark.size;
			shark.eaten = true;
			break;
		}
	}
	return ret;
}

void input() {
	memset(shark_idx, -1, sizeof(shark_idx));
	scanf("%d %d %d", &R, &C, &M);
	sharks.resize(M);

	for (int i = 0; i < M; ++i) {
		int r, c, s, d, z;
		scanf("%d %d %d %d %d", &r, &c, &s, &d, &z);
		--r; --c; --d;
		if (d <= 1) {
			s = s % (R * 2 - 2);
		}
		else {
			s = s % (C * 2 - 2);
		}
		sharks[i] = { r, c, d, s, z, false };
		shark_idx[r][c] = i;
	}
}
```
</details>

### 설명
구현 문제.

1. 최대 100번의 반복을 하므로, 첫 상어 개수를 매 반복마다 순회한다고 해도 최대 O(100 x 100 x 100) = O(1,000,000) 이므로 시간 안에 수행 가능하다.

2. 따라서 Info 구조체 안에 bool eaten 변수를 사용하여 구현해도 된다는 것을 알 수 있다. 
   (안쓰면 매번 먹힌 상어를 지워줘야 함.)

***

## 17140. 이차원 배열과 연산
> https://www.acmicpc.net/problem/17140

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
#include <algorithm>
using namespace std;

int r, c, k, A[100][100], cnt[101], r_num, c_num;
void input();

int max_row;
void sort_row(int y);

int max_col;
void sort_col(int x);


int main() {
	input();
	if (A[r][c] == k) {
		printf("0");
		return 0;
	}

	for (int t = 1; t <= 100; ++t) {
		if (r_num >= c_num) {
			max_col = 0;
			for (int y = 0; y < r_num; ++y) {
				sort_row(y);
			}
			c_num = max_col;
		}
		else {
			max_row = 0;
			for (int x = 0; x < c_num; ++x) {
				sort_col(x);
			}
			r_num = max_row;
		}

		if (A[r][c] == k) {
			printf("%d", t);
			return 0;
		}
	}

	printf("-1");
	return 0;
}

void sort_col(int x) {
	memset(cnt, 0, sizeof(cnt));
	for (int y = 0; y < r_num; ++y) {
		++cnt[A[y][x]];
		A[y][x] = 0;
	}
	vector<pair<int, int>> sorted;

	for (int num = 1; num <= 100; ++num) {
		if (cnt[num]) {
			sorted.push_back({ cnt[num], num });
		}
	}
	sort(sorted.begin(), sorted.end());

	int idx = 0;
	for (auto& p : sorted) {
		A[idx++][x] = p.second;
		A[idx++][x] = p.first;
		if (idx == 100) break;
	}

	max_row = max(max_row, idx);
}

void sort_row(int y) {
	memset(cnt, 0, sizeof(cnt));
	for (int x = 0; x < c_num; ++x) {
		++cnt[A[y][x]];
		A[y][x] = 0;
	}
	vector<pair<int, int>> sorted;

	for (int num = 1; num <= 100; ++num) {
		if (cnt[num]) {
			sorted.push_back({ cnt[num], num });
		}
	}
	sort(sorted.begin(), sorted.end());

	int idx = 0;
	for (auto& p : sorted) {
		A[y][idx++] = p.second;
		A[y][idx++] = p.first;
		if (idx == 100) break;
	}

	max_col = max(max_col, idx);
}

void input() {
	scanf("%d %d %d", &r, &c, &k);
	--r; --c;
	for (int i = 0; i < 3; ++i) {
		for (int j = 0; j < 3; ++j) {
			scanf("%d", &A[i][j]);
		}
	}
	r_num = 3;
	c_num = 3;
}
```
</details>

### 설명
1. 쉬운 구현 문제
2. 반복문을 돌리기 전, 최초에 A[r][c] == k 인 경우를 빼먹으면 안된다.

***

## 17142. 연구소 3
> https://www.acmicpc.net/problem/17142

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
#include <algorithm>
using namespace std;

#define INF 987654321

struct Pos {
	int y, x;
};
const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

Pos viruses[2500];
int N, M, v_num;
char board[50][50];
void input();

int chosed[10];
void dfs(int idx, int from, int num);

int min_time = INF;
int dist[50][50];
void calc();

int main() {
	input();
	dfs(0, 0, M);
	printf("%d", (min_time == INF ? -1 : min_time));
}

void calc() {
	memset(dist, -1, sizeof(dist));
	queue<Pos> q;
	int max_time = 0;

	for (int i = 0; i < M; ++i) {
		Pos& v = viruses[chosed[i]];
		q.push(v);
		dist[v.y][v.x] = 0;
	}

	while (!q.empty()) {
		Pos here = q.front(); q.pop();

		for (int d = 0; d < 4; ++d) {
			int yy = here.y + dy[d], xx = here.x + dx[d];
			if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
			if (board[yy][xx] == 1 || dist[yy][xx] != -1) continue;
			dist[yy][xx] = dist[here.y][here.x] + 1;
			q.push({ yy,xx });
		}
	}

	for (int y = 0; y < N; ++y) {
		for (int x = 0; x < N; ++x) {
			if (board[y][x] != 0) continue;
			if (dist[y][x] == -1) {
				return;
			}
			max_time = max(max_time, dist[y][x]);
		}
	}

	min_time = min(min_time, max_time);
}

void dfs(int idx, int from, int num) {
	if (num == 0) {
		calc();
		return;
	}

	for (int here = from; here <= v_num - num; ++here) {
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
				viruses[v_num++] = { i, j };
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
#include <cstdio>
#include <vector>
using namespace std;

const int dy[4] = { 0, 0, -1, 1 }, dx[4] = { 1, -1, 0, 0 };
struct Info {
    int y, x, d, idx;
};
int N, K;
char color[12][12];
Info horses[10];
vector<int> horse_nums[12][12];
void input();

bool move(int idx);
bool move_white(int idx);
bool move_red(int idx);

void print(int num) {
    printf("\n");
    printf("move %d\n", num);
    for (int y = 0; y < N; ++y) {
        for (int x = 0; x < N; ++x) {
            printf("%d ", horse_nums[y][x].size());
        }
        printf("\n");
    }
}

int main() {
    input();
    for (int turn = 1; turn <= 1000; ++turn) {
        for (int h = 0; h < K; ++h) {
            if (!move(h)) {
                printf("%d", turn);
                return 0;
            }
        }
    }
    printf("-1");
    return 0;
}

bool move(int idx) {
    Info& h = horses[idx];
    int yy = h.y + dy[h.d], xx = h.x + dx[h.d];
    if (yy < 0 || yy >= N || xx < 0 || xx >= N || color[yy][xx] == 2) { // 경계 밖 or 파란색
        h.d = (h.d <= 1 ? 1 - h.d : 5 - h.d);
        yy = h.y + dy[h.d], xx = h.x + dx[h.d];
        if (yy < 0 || yy >= N || xx < 0 || xx >= N || color[yy][xx] == 2) {
            return true;
        }
        else if (color[yy][xx] == 0) {
            return move_white(idx);
        }
        else {
            return move_red(idx);
        }
    }
    else if (color[yy][xx] == 0) { // 흰색
        return move_white(idx);
    }
    else { // 빨간색
        return move_red(idx);
    }
}

bool move_white(int idx) {
    Info& h = horses[idx];
    int y = h.y, x = h.x, yy = h.y + dy[h.d], xx = h.x + dx[h.d];

    vector<int>& from = horse_nums[y][x];
    vector<int>& to = horse_nums[yy][xx];

    int start = h.idx;
    for (int i = start; i < from.size(); ++i) {
        horses[from[i]].y = yy;
        horses[from[i]].x = xx;
        horses[from[i]].idx = to.size();
        to.push_back(from[i]);
    }

    from.erase(from.begin() + start, from.end());

    return horse_nums[yy][xx].size() < 4;
}

bool move_red(int idx) {
    Info& h = horses[idx];
    int y = h.y, x = h.x, yy = h.y + dy[h.d], xx = h.x + dx[h.d];

    vector<int>& from = horse_nums[y][x];
    vector<int>& to = horse_nums[yy][xx];

    int start = h.idx;
    for (int i = from.size() - 1; i >= start; --i) {
        horses[from[i]].y = yy;
        horses[from[i]].x = xx;
        horses[from[i]].idx = to.size();
        to.push_back(from[i]);
    }

    from.erase(from.begin() + start, from.end());

    return horse_nums[yy][xx].size() < 4;
}

void input() {
    scanf("%d %d", &N, &K);
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            scanf("%hhd", &color[i][j]);
        }
    }
    for (int i = 0; i < K; ++i) {
        int y, x, d;
        scanf("%d %d %d", &y, &x, &d);
        --y; --x; --d;
        horses[i] = { y, x, d, 0 };
        horse_nums[y][x].push_back(i);
    }
}
```
</details>

### 설명
1. 말은 번호 순서대로 움직이므로 말들의 정보를 vector에 일렬로 저장한다.

2. 보드를 나타내는 2차원 배열 위에서는, 해당 칸에 존재하는 말들의 vector 상의 인덱스를 저장한다.

***

## 17822. 원판 돌리기
> https://www.acmicpc.net/problem/17822

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <queue>
#include <vector>
using namespace std;

struct Pos {
	int y, x;
};
const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };
int N, M, T, circle[51][50];
void input();

void rotate(int num, int dir, int cnt);

bool discovered[51][50];
bool calc1();
void calc2();

void print() {
	printf("\n");
	for (int num = 1; num <= N; ++num) {
		for (int i = 0; i < M; ++i) {
			printf("%d ", circle[num][i]);
		}
		printf("\n");
	}
}

int main() {
	input();
	while (T--) {
		int x, d, k;
		scanf("%d %d %d", &x, &d, &k);
		for (int num = 1; num <= N; ++num) {
			if (num % x == 0) {
				rotate(num, d, k);
			}
		}
		if (!calc1()) {
			calc2();
		}
	}

	int sum = 0;
	for (int num = 1; num <= N; ++num) {
		for (int i = 0; i < M; ++i) {
			sum += circle[num][i];
		}
	}
	printf("%d", sum);

	return 0;
}

void calc2() {
	double sum = 0;
	int cnt = 0;
	for (int num = 1; num <= N; ++num) {
		for (int i = 0; i < M; ++i) {
			if (circle[num][i]) {
				sum += circle[num][i];
				++cnt;
			}
		}
	}

	sum /= cnt;

	for (int num = 1; num <= N; ++num) {
		for (int i = 0; i < M; ++i) {
			if (circle[num][i]) {
				if (circle[num][i] > sum) {
					--circle[num][i];
				}
				else if(circle[num][i] < sum) {
					++circle[num][i];
				}
			}
		}
	}
}

bool bfs(int num, int i) {
	queue<Pos> q;
	vector<Pos> v;

	discovered[num][i] = true;
	q.push({ num, i });
	    
	while (!q.empty()) {
		Pos here = q.front(); q.pop();
		v.push_back(here);
		for (int d = 0; d < 4; ++d) {
			int yy = here.y + dy[d], xx = here.x + dx[d];
			if (xx == -1) xx = M - 1;
			else if (xx == M) xx = 0;
			if (yy <= 0 || yy > N || discovered[yy][xx]) continue;
			if (circle[here.y][here.x] == circle[yy][xx]) {
				discovered[yy][xx] = true;
				q.push({ yy, xx });
			}
		}
	}

	if (v.size() == 1) return false;

	for (Pos& p : v) {
		circle[p.y][p.x] = 0;
	}

	return true;
}

bool calc1() {
	memset(discovered, false, sizeof(discovered));
	bool check = false;

	for (int num = 1; num <= N; ++num) {
		for (int i = 0; i < M; ++i) {
			if (!discovered[num][i] && circle[num][i] != 0) {
				if (bfs(num, i)) {
				check = true;
				}
			}
		}
	}

	return check;
}

void rotate(int num, int dir, int cnt) {
	int tmp[50] = { 0, };

	if (dir == 0) {
		for(int i = 0; i < M; ++i) {
			tmp[(i + cnt) % M] = circle[num][i];
		}
	}
	else {
		for (int i = 0; i < M; ++i) {
			tmp[(i - cnt + M) % M] = circle[num][i];
		}
	}

	memcpy(circle[num], tmp, sizeof(tmp));
}

void input() {
	scanf("%d %d %d", &N, &M, &T);
	for (int i = 1; i <= N; ++i) {
		for (int j = 0; j < M; ++j) {
			scanf("%d", &circle[i][j]);
		}
	}
}
```
</details>

### 설명
구현 + bfs 문제.

지운 수를 0으로 정의하였으므로, 값이 0인 부분에서 bfs를 수행하지 않도록 해야한다.

bfs / dfs 에서 다음 노드로 탐색을 파고 들어갈 때, 탐색 조건을 만족하는지 꼭 확인해야한다.

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
#include <cstdio>
#include <cstring>
#include <vector>
#include <algorithm>
using namespace std;

const int dy[8] = { -1, -1, 0, 1, 1, 1, 0, -1 }, dx[8] = { 0, -1, -1, -1, 0, 1, 1, 1 };

struct Pos {
  int y, x, d;
  bool eaten;
};

int f_num;
struct Info {
  char board[4][4];
  Pos fishes[17];
  int sy, sx, sd;
};

Info info;
void input();

int max_sum;
void dfs(int sum);

void print() {
  for (int y = 0; y < 4; ++y) {
    for (int x = 0; x < 4; ++x) {
      printf("%3d", info.board[y][x]);
    }
    printf("\n");
  }
}

int main() {
  input();

  // (0, 0) 물고기 먹기
  int num = info.board[0][0];
  info.board[0][0] = 0;
  info.sy = info.sx = 0;
  info.sd = info.fishes[num].d;
  info.fishes[num].eaten = true;
  // dfs 시작
  dfs(num);

  printf("%d", max_sum);

  return 0;
}

void move_fish(int num) {
  Pos& fish = info.fishes[num];
  int y = fish.y, x = fish.x;
  for (int i = 0; i < 8; ++i) {
    int yy = y + dy[fish.d];
    int xx = x + dx[fish.d];
    if (yy < 0 || yy >= 4 || xx < 0 || xx >= 4 || (yy == info.sy && xx == info.sx)) {
      fish.d = (fish.d + 1) % 8;
      continue;
    }
    Pos& _fish = info.fishes[info.board[yy][xx]];
    swap(info.board[y][x], info.board[yy][xx]);
    fish.y = yy; fish.x = xx;
    _fish.y = y; _fish.x = x;
    break;
  }
}

void dfs(int sum) {
  max_sum = max(max_sum, sum);

  // dfs 시작 전 info 값 백업
  //Info ori = info;
  Info* ori = new Info();
  memcpy(ori, &info, sizeof(info));

  // 물고기 이동
  for (int num = 1; num <= 16; ++num) {
    if (info.fishes[num].eaten) continue;
    move_fish(num);
  }

  // 물고기 이동한 상태 백업
  //Info moved = info;
  Info* moved = new Info();
  memcpy(moved, &info, sizeof(info));

  // 먹을 수 있는 물고기 찾기
  vector<pair<int, int>> cand;
  for (int i = 1; ; ++i) {
    int yy = info.sy + dy[info.sd] * i;
    int xx = info.sx + dx[info.sd] * i;
    if (yy < 0 || yy >= 4 || xx < 0 || xx >= 4) break;
    if (info.board[yy][xx] != 0) {
      cand.emplace_back(yy, xx);
    }
  }

  // 먹을 수 있는 물고기 없으면 탐색 종료
  if (cand.empty()) {
    info = *ori;
    delete ori;
    delete moved;
    return;
  }

  // 물고기 먹고 재귀호출
  for (auto& p : cand) {
    int eaten_num = info.board[p.first][p.second];
    info.board[p.first][p.second] = 0;
    Pos& eaten_fish = info.fishes[eaten_num];
    eaten_fish.eaten = true;
    info.sy = p.first;
    info.sx = p.second;
    info.sd = eaten_fish.d;
    dfs(sum + eaten_num);

    // info 값 물고기만 이동시킨 상태로 복구
    info = *moved;
  }

  // info 값 복구
  info = *ori;
  delete ori;
  delete moved;
}

void input() {
  for (int y = 0; y < 4; ++y) {
    for (int x = 0; x < 4; ++x) {
      int a, b;
      scanf("%d %d", &a, &b);
      info.board[y][x] = a;
      info.fishes[a] = { y, x, b - 1, false };
    }
  }
}
```
</details>

### 설명
1. dfs로 가능한 모든 경우를 탐색하며 최댓값을 찾는 문제.

2. 적절한 print 함수를 만들어서, 한 기능이 완성되면 테스트하여 디버깅하며 구현해야 한다.

3. dfs 함수 내부에서 구조체 원본을 백업해놓을때, 우선적으로는 스택에 할당하되, 구현이 끝나고 나서 힙에 할당하는 방식으로 바꾸면 스택 공간을 절약할 수 있다.

   new / delete 연산자로 동적 할당 / 메모리 해제 를 할 수 있다.

***

## 19237. 어른 상어
> https://www.acmicpc.net/problem/19237

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
using namespace std;

struct Smell {
    int num, left;
};

struct Info {
    int y, x, d;
    bool out;
};

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

Smell smell[20][20];

int dir[401][4][4]; // 위, 아래, 왼, 오
int N, M, K;
int shark_num[20][20], tmp[20][20], shark_cnt;
Info sharks[401];
void input();

void print() {
    printf("<shark_num>\n");
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            printf("%d ", shark_num[i][j]);
        }
        printf("\n");
    }
    printf("<smell>\n");
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            printf("[%d %d] ", smell[i][j].num, smell[i][j].left);
        }
        printf("\n");
    }
}

void print2() {
    for (int num = 1; num <= M; ++num) {
        printf("%d: [%d, %d], dir: %d\n", num, sharks[num].y, sharks[num].x, sharks[num].d);
    }
}

void move_shark(int num);

int main() {
    input();
    for (int t = 1; t <= 1000; ++t) {
        // 1. 상어 이동
        for (int num = 1; num <= M; ++num) {
            if (sharks[num].out) continue;
            move_shark(num);
        }
        
        // 2. shark_num 배열 갱신
        memset(tmp, 0, sizeof(tmp));
        for (int num = 1; num <= M; ++num) {
            Info& shark = sharks[num];
            if (shark.out) continue;
            if(tmp[shark.y][shark.x] == 0) {
                tmp[shark.y][shark.x] = num;
            }
            else if (tmp[shark.y][shark.x] > num) {
                sharks[tmp[shark.y][shark.x]].out = true;
                tmp[shark.y][shark.x] = num;
                --shark_cnt;
            }
            else if (tmp[shark.y][shark.x] < num) {
                sharks[num].out = true;
                --shark_cnt;
            }
        }
        memcpy(shark_num, tmp, sizeof(shark_num));

        // 1번 상어만 남았는지 확인
        if (shark_cnt == 1) {
            printf("%d", t);
            return 0;
        }
        
        // 3. 냄새 뿌리기
        for (int y = 0; y < N; ++y) {
            for (int x = 0; x < N; ++x) {
                if (shark_num[y][x]) {
                    smell[y][x] = { shark_num[y][x], K + 1 };
                }
            }
        }

        // 4. 냄새 1 줄이기
        for (int y = 0; y < N; ++y) {
            for (int x = 0; x < N; ++x) {
                if (smell[y][x].left) {
                    --smell[y][x].left;
                }
            }
        }

    }

    printf("-1");
    return 0;
}

void move_shark(int num) {
    Info& shark = sharks[num];
    Info candi = { -1, };

    int y = shark.y, x = shark.x;

    // 1. 빈 칸 찾기
    for (int i = 0; i < 4; ++i) {
        int d = dir[num][shark.d][i];
        int yy = y + dy[d];
        int xx = x + dx[d];
        if (yy < 0 || yy >= N || xx < 0 || xx >= N || smell[yy][xx].left > 0) {
            continue;
        }
        candi = { yy, xx, d };
        break;
    }

    // 2. 빈칸 있을 때
    if (candi.y != -1) {
        shark.y = candi.y;
        shark.x = candi.x;
        shark.d = candi.d;
        return;
    }

    // 3. 빈칸 없을 때
    for (int i = 0; i < 4; ++i) {
        int d = dir[num][shark.d][i];
        int yy = y + dy[d];
        int xx = x + dx[d];
        if (yy < 0 || yy >= N || xx < 0 || xx >= N) {
            continue;
        }
        if (smell[yy][xx].num == num) {
            candi = { yy, xx, d };
            break;
        }
    }


    if (candi.y != -1) {
        shark.y = candi.y;
        shark.x = candi.x;
        shark.d = candi.d;
    }
}

void input() {
    scanf("%d %d %d", &N, &M, &K);
    shark_cnt = M;
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            scanf("%d", &shark_num[i][j]);
            if (shark_num[i][j] > 0) {
                sharks[shark_num[i][j]] = { i, j, 0, false };
            }
        }
    }
    for (int num = 1; num <= M; ++num) {
        int d; scanf("%d", &d);
        sharks[num].d = d - 1;
    }
    for (int num = 1; num <= M; ++num) {
        for (int d = 0; d < 4; ++d) {
            for (int i = 0; i < 4; ++i) {
                int to; scanf("%d", &to);
                dir[num][d][i] = to - 1;
            }
        }
    }
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            if (shark_num[i][j]) {
                smell[i][j] = { shark_num[i][j], K };
            }
            else {
                smell[i][j].left = 0;
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
```
</details>

### 설명
구현 + bfs 문제.

1. 문제에서 필요한 정보 (좌표, 두 좌표를 갖는 손님) 를 구조체로 표현하여 코드를 작성하면 깔끔하다.

2. **문제를 푸는 도중도중에 반드시 print 해가며 올바르게 진행중인지 확인해야한다.**

***

## 20055. 컨베이어 벨트 위의 로봇
> https://www.acmicpc.net/problem/20055

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>

int N, K, A[200], zero_cnt;
bool robot[200];
void input();

void rotate();
void move_robots();

int main() {
  input();

  for (int turn = 1; ; ++turn) {
    // 1. 컨베이어 벨트 및 로봇 회전
    rotate();
    // 2. 내리는 위치에서 로봇 내리기
    if (robot[N - 1]) robot[N - 1] = false;
    // 3. 로봇 움직이기
    move_robots();
    // 4. 내리는 위치에서 로봇 내리기
    if (robot[N - 1]) robot[N - 1] = false;
    // 5. 로봇 올리기
    if (A[0] > 0 && !robot[0]) {
      --A[0];
      robot[0] = true;
      if (A[0] == 0) {
        ++zero_cnt;
      }
    }
    if (zero_cnt >= K) {
      printf("%d", turn);
      break;
    }
  }

  return 0;
}

void move_robots() {
  for (int r = N - 2; r >= 0; --r) {
    if (robot[r] && !robot[r + 1] && A[r + 1] >= 1) {
      robot[r] = false;
      robot[r + 1] = true;
      --A[r + 1];
      if (A[r + 1] == 0) ++zero_cnt;
    }
  }
}

void rotate() {
  int tmp = A[2 * N - 1];
  for (int i = 2 * N - 1; i >= 1; --i) {
    A[i] = A[i - 1];
  }
  A[0] = tmp;

  for (int i = N - 1; i >= 1; --i) {
    robot[i] = robot[i - 1];
  }
  robot[0] = false;
}

void input() {
  scanf("%d %d", &N, &K);
  for (int i = 0; i < 2 * N; ++i) {
    scanf("%d", &A[i]);
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
#include <cstdio>
#include <cstring>
#include <vector>
using namespace std;

const int dy[8] = { -1, -1, 0, 1, 1, 1, 0, -1 }, dx[8] = { 0, 1, 1, 1, 0, -1, -1, -1 };
struct Ball {
  int y, x, m, s, d;
};
struct Grid {
  int m_sum, s_sum, b_cnt, dir;
  bool diff;
};

int N, M, K;
vector<Ball> balls;
Grid board[50][50];
void input();

void move_ball(int num);
void update_balls(int y, int x);

int main() {
  input();

  for (int order = 0; order < K; ++order) {
    // 1. 모든 파이어볼 이동
    memset(board, 0, sizeof(board));
    for (int num = 0; num < balls.size(); ++num) {
      move_ball(num);
    }
    // 2. 파이어볼 벡터 갱신
    balls.clear();
    for (int y = 0; y < N; ++y) {
      for (int x = 0; x < N; ++x) {
        update_balls(y, x);
      }
    }
  }

  int ret = 0;
  for (Ball& b : balls) {
    ret += b.m;
  }
  printf("%d", ret);

  return 0;
}

void update_balls(int y, int x) {
  Grid& g = board[y][x];
  if (g.b_cnt == 0) return;
  if (g.b_cnt == 1) {
    balls.push_back({ y, x, g.m_sum, g.s_sum, g.dir });
  }
  else {
    int m = g.m_sum / 5;
    int s = g.s_sum / g.b_cnt;
    if (m == 0) return;
    if (g.diff) {
      for (int d = 1; d <= 7; d += 2) {
        balls.push_back({ y, x, m, s, d });
      }
    }
    else {
      for (int d = 0; d <= 6; d += 2) {
        balls.push_back({ y, x, m, s, d });
      }
    }
  }
}

void move_ball(int num) {
  Ball& b = balls[num];
  int yy = b.y + dy[b.d] * b.s, xx = b.x + dx[b.d] * b.s;
  while (yy < 0) yy += N; while (yy >= N) yy -= N;
  while (xx < 0) xx += N; while (xx >= N) xx -= N;
  board[yy][xx].m_sum += b.m;
  board[yy][xx].s_sum += b.s;
  ++board[yy][xx].b_cnt;
  if (board[yy][xx].b_cnt == 1) {
    board[yy][xx].dir = b.d;
  }
  else {
    if (board[yy][xx].dir % 2 != b.d % 2) {
      board[yy][xx].diff = true;
    }
  }
}

void input() {
  scanf("%d %d %d", &N, &M, &K);
  balls.resize(M);
  for (int i = 0; i < M; ++i) {
    scanf("%d %d %d %d %d", &balls[i].y, &balls[i].x, &balls[i].m, &balls[i].s, &balls[i].d);
    --balls[i].y; --balls[i].x;
    //balls[i].s %= N;
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
#include <cstdio>

struct Pos {
  int y, x;
};
int N, A[499][499], dir[3] = { 3, 0, 1 }; // d_r, dir, d_l
Pos from, to, to2;
void input();

const int dy[4] = { 0, 1, 0, -1 }, dx[4] = { -1, 0, 1, 0 };
const int d_num[9][3] = { // d_r, dir, d_l
  {1, 0, 0}, {1, 1, 0}, {2, 1, 0}, {1, 2, 0}, {0, 3, 0}, {0, 0, 1}, {0, 1, 1}, {0, 2, 1}, {0, 1, 2}
};
const int per[9] = { 1, 7, 2, 10, 5, 1, 7, 10, 2 };

int out;
void blow(int len);

int main() {
  input();

  for (int len = 1; ; ++len) {
    blow(len);
    for (int i = 0; i < 3; ++i) {
      dir[i] = (dir[i] + 1) % 4;
    }

    blow(len);
    for (int i = 0; i < 3; ++i) {
      dir[i] = (dir[i] + 1) % 4;
    }

    if (len == N - 1) {
      blow(len);
      break;
    }
  }

  printf("%d", out);

  return 0;
}

void blow(int len) {
  if (len == 0) return;

  to = { from.y + dy[dir[1]], from.x + dx[dir[1]] };
  to2 = { to.y + dy[dir[1]], to.x + dx[dir[1]] };

  int sum = 0;
  for (int i = 0; i < 9; ++i) {
    int yy = from.y, xx = from.x;
    for (int j = 0; j < 3; ++j) {
      yy += dy[dir[j]] * d_num[i][j];
      xx += dx[dir[j]] * d_num[i][j];
    }
    int sand = A[to.y][to.x] * per[i] / 100;
    sum += sand;
    if (yy < 0 || yy >= N || xx < 0 || xx >= N) {
      out += sand;
    }
    else {
      A[yy][xx] += sand;
    }
  }
  A[to.y][to.x] -= sum;

  if (to2.y < 0 || to2.y >= N || to2.x < 0 || to2.x >= N) {
    out += A[to.y][to.x];
  }
  else {
    A[to2.y][to2.x] += A[to.y][to.x];
  }
  A[to.y][to.x] = 0;

  from = to;

  blow(len - 1);
}

void input() {
  scanf("%d", &N);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      scanf("%d", &A[i][j]);
    }
  }
  from = { N / 2, N / 2 };
}
```
</details>

### 설명
2차원 배열 위에서 특정 방향으로의 점 이동을 연습할 수 있는 문제

***

## 20058. 마법사 상어와 파이어스톰
> https://www.acmicpc.net/problem/20058

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
int N, Q, L, A[64][64];
void input();

int tmp[64][64];
void firestorm();
void rotate(int y, int x);

bool discovered[64][64];
int bfs(int y, int x);

int main() {
  input();
  while (Q--) {
    scanf("%d", &L);
    L = (1 << L);
    // 파이어스톰 시전
    firestorm();
  }

  // 얼음 합 출력
  int sum = 0;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      sum += A[y][x];
    }
  }
  printf("%d\n", sum);

  // 덩어리가 차지하는 칸 계산
  int max_cnt = 0;
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      if (!discovered[y][x] && A[y][x] > 0) {
        max_cnt = max(max_cnt, bfs(y, x));
      }
    }
  }
  printf("%d", max_cnt);

  return 0;
}

int bfs(int y, int x) {
  int cnt = 0;
  queue<Pos> q;

  discovered[y][x] = true;
  q.push({ y, x });

  while (!q.empty()) {
    Pos here = q.front(); q.pop();
    ++cnt;
    for (int d = 0; d < 4; ++d) {
      int yy = here.y + dy[d], xx = here.x + dx[d];
      if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
      if (discovered[yy][xx] || A[yy][xx] == 0) continue;
      discovered[yy][xx] = true;
      q.push({ yy, xx });
    }
  }

  return cnt;
}

void firestorm() {
  // 시계방향 90도 회전
  for (int y = 0; y < N; y += L) {
    for (int x = 0; x < N; x += L) {
      rotate(y, x);
    }
  }
  memcpy(A, tmp, sizeof(A));

  // 얼음 양 조절
  memset(tmp, 0, sizeof(tmp));
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      if (A[y][x] == 0) continue;
      int cnt = 0;
      for (int d = 0; d < 4; ++d) {
        int yy = y + dy[d], xx = x + dx[d];
        if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
        if (A[yy][xx] > 0) ++cnt;
      }
      if (cnt < 3) {
        tmp[y][x] = A[y][x] - 1;
      }
      else {
        tmp[y][x] = A[y][x];
      }
    }
  }
  memcpy(A, tmp, sizeof(A));
}

void rotate(int y, int x) {
  for (int yy = 0; yy < L; ++yy) {
    for (int xx = 0; xx < L; ++xx) {
      tmp[y + xx][x + L - 1 - yy] = A[y + yy][x + xx];
    }
  }
}

void input() {
  scanf("%d %d", &N, &Q);
  N = (1 << N);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      scanf("%d", &A[i][j]);
    }
  }
}
```
</details>

### 설명
구현 + bfs 문제.
struct와 전역변수를 적절히 사용하면 간단하게 코드를 작성할 수 있다.

***

## 상어 초등학교
> https://www.acmicpc.net/problem/21608

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>

#define max(a, b) ((a) > (b) ? (a) : (b))

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };
struct Info {
  int num, like[4], y, x;
  bool is_liked(int f_num) {
    for (int i = 0; i < 4; ++i) {
      if (like[i] == f_num) return true;
    }
    return false;
  }
};

int N, student_num[20][20], like_cnt[20][20], max_like_cnt, empty_cnt[20][20], max_empty_cnt;
Info students[400];
void input();

void calc1(Info& s, int y, int x);

int main() {
  input();
  for (Info& s : students) {
    // 1. 각 위치의 좋아하는 인접 학생 수와, 가장 큰 좋아하는 인접 학생 수 구하기
    max_like_cnt = 0;
    for (int y = 0; y < N; ++y) {
      for (int x = 0; x < N; ++x) {
        if (student_num[y][x] != 0) continue;
        calc1(s, y, x);
      }
    }
    // 2. 가장 큰 좋아하는 인접 학생 수를 가진 칸들 중에서, 가장 큰 인접 빈 칸 개수 구하기
    max_empty_cnt = 0;
    for (int y = 0; y < N; ++y) {
      for (int x = 0; x < N; ++x) {
        if (student_num[y][x] != 0 || like_cnt[y][x] != max_like_cnt) continue;
        max_empty_cnt = max(max_empty_cnt, empty_cnt[y][x]);
      }
    }
    // 3. 행 커지는 방향, 열 커지는 방향으로 탐색하면서 조건 만족하는 자리 찾아서 학생 앉히기.
    bool put = false;
    for (int y = 0; y < N; ++y) {
      for (int x = 0; x < N; ++x) {
        if (student_num[y][x] != 0) continue;
        if (like_cnt[y][x] == max_like_cnt && empty_cnt[y][x] == max_empty_cnt) {
          student_num[y][x] = s.num;
          put = true;
          for (int d = 0; d < 4; ++d) {
            int yy = y + dy[d], xx = x + dx[d];
            if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
            --empty_cnt[yy][xx];
          }
          s.y = y; s.x = x;
          break;
        }
      }
      if (put) break;
    }
  }

  // 만족도 계산
  int sum = 0;
  for (Info& s : students) {
    int cnt = 0;
    for (int d = 0; d < 4; ++d) {
      int yy = s.y + dy[d], xx = s.x + dx[d];
      if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
      if (s.is_liked(student_num[yy][xx])) ++cnt;
    }
    if (cnt == 1) sum += 1;
    else if (cnt == 2) sum += 10;
    else if (cnt == 3) sum += 100;
    else if (cnt == 4) sum += 1000;
  }
  printf("%d", sum);

  return 0;
}

void calc1(Info& s, int y, int x) {
  int cnt = 0;
  for (int d = 0; d < 4; ++d) {
    int yy = y + dy[d], xx = x + dx[d];
    if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
    if (student_num[yy][xx] != 0 && s.is_liked(student_num[yy][xx])) {
      ++cnt;
    }
  }
  like_cnt[y][x] = cnt;
  max_like_cnt = max(max_like_cnt, cnt);
}

void input() {
  scanf("%d", &N);
  for (int i = 0; i < N * N; ++i) {
    Info& s = students[i];
    scanf("%d", &s.num);
    for (int j = 0; j < 4; ++j) {
      scanf("%d", &s.like[j]);
    }
  }
  empty_cnt[0][0] = empty_cnt[0][N - 1] = empty_cnt[N - 1][0] = empty_cnt[N - 1][N - 1] = 2;
  for (int i = 1; i < N - 1; ++i) {
    empty_cnt[0][i] = empty_cnt[N - 1][i] = empty_cnt[i][0] = empty_cnt[i][N - 1] = 3;
  }
  for (int i = 1; i < N - 1; ++i) {
    for (int j = 1; j < N - 1; ++j) {
      empty_cnt[i][j] = 4;
    }
  }
}
```
</details>

### 설명
구현 문제.
struct 및, 구조체 함수를 사용하면 간단하게 구현할 수 있다.

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