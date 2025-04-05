# 삼성 SW 역량 테스트 기출 문제 (백준)
## 13460. 구슬 탈출 2
> https://www.acmicpc.net/problem/13460

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

struct Pos {
  int y, x;
  bool operator==(const Pos& rhs) const {
    return y == rhs.y && x == rhs.x;
  }
  bool operator!=(const Pos& rhs) const {
    return !(y == rhs.y && x == rhs.x);
  }
};

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };

int N, M;
char board[10][10];
Pos red, blue, hole;
void input();

int ret = 11;
void dfs(int d);
void lean(int dir);

int main() {
  input();
  dfs(1);
  printf("%d", (ret == 11 ? -1 : ret));
  return 0;
}

void dfs(int d) {
  if (d >= ret) return;
  Pos red_t = red, blue_t = blue;
  for (int dir = 0; dir < 4; ++dir) {
    lean(dir);
    if (red == red_t && blue == blue_t) continue;
    if (blue != hole) {
      if (red == hole) {
        ret = min(ret, d);
        return;
      }
      else {
        dfs(d + 1);
      }
    }
    red = red_t; blue = blue_t;
  }
}

void lean(int dir) {
  bool red_first = dir < 2 ? (dir == 0 ? red.y < blue.y : red.y > blue.y) : (dir == 2 ? red.x < blue.x : red.x > blue.x);

  while (red.y < N - 1 && red.x < M - 1 && board[red.y][red.x] != '#' && red != hole) {
    red.y += dy[dir]; red.x += dx[dir];
  }
  while (blue.y < N - 1 && blue.x < M - 1 && board[blue.y][blue.x] != '#' && blue != hole) {
    blue.y += dy[dir]; blue.x += dx[dir];
  }

  if (blue == hole || red == hole) return;

  red.y -= dy[dir]; red.x -= dx[dir];
  blue.y -= dy[dir]; blue.x -= dx[dir];

  if (red == blue) {
    if (red_first) {
      blue.y -= dy[dir]; blue.x -= dx[dir];
    }
    else {
      red.y -= dy[dir]; red.x -= dx[dir];
    }
  }
}

void input() {
  scanf("%d %d", &N, &M);
  getchar();

  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < M; ++j) {
      char ch; scanf("%c", &ch);
      board[i][j] = ch;
      if (ch == 'R') {
        red = { i, j };
      }
      else if (ch == 'B') {
        blue = { i, j };
      }
      else if (ch == 'O') {
        hole = { i, j };
      }
    }
    getchar();
  }
}
```
</details>

### 설명
1. red와 blue 값을 매개변수로 넘기지 않고, 보드를 기울이기 전에 red와 blue 값을 저장해두면 전역변수로 사용할 수 있다. 

   => dfs 함수에 매개변수로 넘기지 않고 깔끔하게 작성 가능하다.
2. 보드를 같은 방향으로 연속해서 두 번 이상 기울이는건 의미 없다. 

   => 가지치기.

***

## 12100. 2048 (Easy)
> https://www.acmicpc.net/problem/12100

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>

#define max(a, b) ((a) > (b) ? (a) : (b))

struct Info {
  int board[20][20];
  void rotate();
  void move();
  void calc();
};
int tmp[20][20];

int N;
Info info;
void input();

int ret;
void dfs(int depth);

int main() {
  input();
  dfs(0);
  printf("%d", ret);
  return 0;
}

void dfs(int depth) {
  if (depth == 5) {
    info.calc();
    return;
  }
  Info saved = info;
  for (int i = 0; i < 4; ++i) {
    info = saved;
    info.move();
    dfs(depth + 1);
    saved.rotate();
  }
}

void Info::calc() {
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      ret = max(ret, board[i][j]);
    }
  }
}

void Info::move() {
  for (int x = 0; x < N; ++x) {
    int btm = N - 1;
    for (int y = N - 2; y >= 0; --y) {
      if (board[y][x] == 0) continue;
      if (board[btm][x] == 0) {
        int n = board[y][x];
        board[y][x] = 0;
        board[btm][x] = n;
      }
      else if (board[btm][x] == board[y][x]) {
        board[btm--][x] *= 2;
        board[y][x] = 0;
      }
      else {
        int n = board[y][x];
        board[y][x] = 0;
        board[--btm][x] = n;
      }
    }
  }
}

void Info::rotate() {
  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < N; ++x) {
      tmp[x][N - 1 - y] = board[y][x];
    }
  }
  memcpy(board, tmp, sizeof(board));
}

void input() {
  scanf("%d", &N);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < N; ++j) {
      scanf("%d", &info.board[i][j]);
    }
  }
}
```
</details>

### 설명
1. 가장 큰 블록만을 출력하면 되는 문제이므로, 보드를 회전하여도 상관 없다. 따라서 보드를 회전시키며 블록을 이동하면, 한 쪽 방향으로 이동하는 부분만 구현하면 된다.
2. Info 값을 매개변수로 넘기는게 아니라, 전역변수로 설정하고 재귀호출 전에 미리 저장해놓는 방식으로 구현하면 코드가 훨씬 깔끔해진다.
3. 보드를 Info struct 에 집어넣고, 보드의 회전, 블록의 이동, 가장 큰 블록 구하기 연산을 구조체 함수로 구현하면 코드가 깔끔하다.

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
#include <cstdio>

const int dy[4] = { 0, 0, -1, 1 }, dx[4] = { 1, -1, 0, 0 };
int N, M, K, y, x, d;
char dice[6]; //  (주사위 면) 0: 위, 1: 아래, 2: 앞, 3: 오, 4: 뒤, 5: 왼
char to[6][4]; // (주사위 면 / 방향) 0: 오, 1: 왼, 2: 위, 3: 아
char board[20][20];
void input();

void init();
void move();

int main() {
  init();
  input();
  while (K--) {
    scanf("%d", &d); --d;
    move();
  }
  return 0;
}

void move() {
  // 주사위 좌표 이동
  int yy = y + dy[d], xx = x + dx[d];
  if (yy < 0 || yy >= N || xx < 0 || xx >= M) return;
  y = yy; x = xx;

  // 주사위 회전
  char tmp[6];
  for (int i = 0; i < 6; ++i) {
    tmp[to[i][d]] = dice[i];
  }
  for (int i = 0; i < 6; ++i) {
    dice[i] = tmp[i];
  }

  // 칸과 상호작용
  if (board[y][x] == 0) {
    board[y][x] = dice[1];
  }
  else {
    dice[1] = board[y][x];
    board[y][x] = 0;
  }

  printf("%hhd\n", dice[0]);
}

void init() {
  // to[i][j] = k
  // i: 0위 1아 2앞 3오 4뒤 5왼 (주사위 면)
  // j: 0오 1왼 2위 3아 (이동 방향)
  // k: 0위 1아 2앞 3오 4뒤 5왼 (주사위 면) 
  to[0][0] = 3; to[0][1] = 5; to[0][2] = 4; to[0][3] = 2;
  to[1][0] = 5; to[1][1] = 3; to[1][2] = 2; to[1][3] = 4;
  to[2][0] = 2; to[2][1] = 2; to[2][2] = 0; to[2][3] = 1;
  to[3][0] = 1; to[3][1] = 0; to[3][2] = 3; to[3][3] = 3;
  to[4][0] = 4; to[4][1] = 4; to[4][2] = 1; to[4][3] = 0;
  to[5][0] = 0; to[5][1] = 1; to[5][2] = 5; to[5][3] = 5;
}

void input() {
  scanf("%d %d %d %d %d", &N, &M, &y, &x, &K);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < M; ++j) {
      scanf("%hhd", &board[i][j]);
    }
  }
}
```
</details>

### 설명
1. 주석을 잘 활용하자.
2. char 변수는 scanf에서는 %hhd, printf에서는 %d 사용한다.
3. unsigned char 변수는 scanf 에서는 %hhu, printf에서는 %u 사용한다.
4. unsigned int 변수는 scanf 에서는 %u, printf에서는 %u 사용한다.

***

## 14500: 테트로미노
> https://www.acmicpc.net/problem/14500

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
#include <algorithm>
using namespace std;

int N, M, board[500][500];
bool basic[7][4][4] = {
  {{1, 1, 1, 1}},
  {{1, 1}, {1, 1}},
  {{1}, {1}, {1, 1}},
  {{0, 1}, {0, 1}, {1, 1}},
  {{1}, {1, 1}, {0, 1}},
  {{0, 1}, {1, 1}, {1}},
  {{1, 1, 1}, {0, 1}}
};
vector<vector<pair<int, int>>> shapes;
void input();

int calc(int idx);

int main() {
  input();
  int ret = 0;
  for (int i = 0; i < shapes.size(); ++i) {
    ret = max(ret, calc(i));
  }
  printf("%d", ret);
  return 0;
}

int calc(int idx) {
  auto& shape = shapes[idx];
  int ret = 0;

  for (int y = 0; y < N; ++y) {
    for (int x = 0; x < M; ++x) {
      int sum = 0;
      for (auto& p : shape) {
        int yy = y + p.first, xx = x + p.second;
        if (yy < 0 || yy >= N || xx < 0 || xx >= M) {
          sum = 0;
          break;
        }
        sum += board[yy][xx];
      }
      ret = max(ret, sum);
    }
  }

  return ret;
}

void rotate(int idx) {
  bool tmp[4][4];
  for (int y = 0; y < 4; ++y) {
    for (int x = 0; x < 4; ++x) {
      tmp[x][3 - y] = basic[idx][y][x];
    }
  }
  memcpy(basic[idx], tmp, sizeof(tmp));
}

void add_shape(int idx, int rot) { // basic[idx] 도형을 회전시키며 rot번 추가
  for (int r = 0; r < rot; ++r) {
    shapes.emplace_back();
    int yy = -1, xx = -1;
    for (int y = 0; y < 4; ++y) {
      for (int x = 0; x < 4; ++x) {
        if (basic[idx][y][x]) {
          if (yy == -1) {
            yy = y; xx = x;
          }
          shapes.back().push_back({ y - yy, x - xx });
        }
      }
    }
    rotate(idx);
  }
}

void input() {
  scanf("%d %d", &N, &M);
  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < M; ++j) {
      scanf("%d", &board[i][j]);
    }
  }

  add_shape(0, 2);
  add_shape(1, 1);
  for (int i = 2; i < 7; ++i) {
    add_shape(i, 4);
  }
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
```
</details>

### 설명
N값이 최대 15이므로, 모든 경우를 다 세어도 2^15(약 32000) 밖에 되지 않는다.

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
```
</details>

### 설명
dfs로 연산자 선택하는 문제.

***

## 14889. 스타트와 링크
> https://www.acmicpc.net/problem/14889

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#define min(a, b) ((a) < (b) ? (a) : (b))

int N;
int score[20][20];
void input();

bool chosed[20];
int ret = 987654321;
void dfs(int idx, int num);

int main() {
	input();
	dfs(0, N / 2);
	printf("%d", ret);
	return 0;
}

void calc() {
	int A[10], B[10], a = 0, b = 0;
	for (int i = 0; i < N; ++i) {
		if (chosed[i]) A[a++] = i;
		else B[b++] = i;
	}

	int score_a = 0, score_b = 0;
	for (int i = 0; i < N / 2; ++i) {
		for (int j = 0; j < N / 2; ++j) {
			score_a += score[A[i]][A[j]];
			score_b += score[B[i]][B[j]];
		}
	}

	ret = min(ret, (score_a > score_b ? score_a - score_b : score_b - score_a));
}

void dfs(int idx, int num) {
	if (num == 0) {
		calc();
		return;
	}
	for (int here = idx; here <= N - 1 - num; ++here) {
		chosed[here] = true;
		dfs(here + 1, num - 1);
		chosed[here] = false;
	}
}

void input() {
	scanf("%d", &N);
	for (int i = 0; i < N; ++i) {
		for (int j = 0; j < N; ++j) {
			scanf("%d", &score[i][j]);
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
#include <cstdio>

int N, L, board[100][100];
void input();

bool calc_row(int y);
bool calc_col(int x);

int main() {
	input();
	int ret = 0;
	for (int i = 0; i < N; ++i) {
		if (calc_col(i)) {
			++ret;
		}
		if (calc_row(i)) {
			++ret;
		}
	}
	printf("%d", ret);
	return 0;
}

bool calc_row(int y) {
	int range = -1;
	for (int x = 0; x < N - 1; ++x) {
		int lx, rx;
		if (board[y][x] == board[y][x + 1]) {
			continue;
		}
		else if (board[y][x] + 1 == board[y][x + 1]) {
			lx = x - L + 1;
			rx = x;
			if (lx <= range) return false;
		}
		else if (board[y][x] == board[y][x + 1] + 1) {
			lx = x + 1;
			rx = x + L;
			if (rx >= N) return false;
		}
		else {
			return false;
		}
		bool same_h = true;
		for (int i = lx; i < rx; ++i) {
			if (board[y][i] != board[y][i + 1]) {
				same_h = false;
				break;
			}
		}
		if (same_h) {
			range = rx;
		}
		else {
			return false;
		}
	}
	return true;
}
bool calc_col(int x) {
	int range = -1;
	int ly, ry;
	for (int y = 0; y < N - 1; ++y) {
		if (board[y][x] == board[y + 1][x]) {
			continue;
		}
		else if (board[y][x] + 1 == board[y + 1][x]) {
			ly = y - L + 1;
			ry = y;
			if (ly <= range) return false;
		}
		else if (board[y][x] == board[y + 1][x] + 1) {
			ly = y + 1;
			ry = y + L;
			if (ry >= N) return false;
		}
		else {
			return false;
		}
		bool same_h = true;
		for (int i = ly; i < ry; ++i) {
			if (board[i][x] != board[i + 1][x]) {
				same_h = false;
				break;
			}
		}
		if (same_h) {
			range = ry;
		}
		else {
			return false;
		}
	}
	return true;
}

void input() {
	scanf("%d %d", &N, &L);
	for (int i = 0; i < N; ++i) {
		for (int j = 0; j < N; ++j) {
			scanf("%d", &board[i][j]);
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
#include <cstdio>
#include <cstring>

int K;
char wheel[4][9];

bool rotated[4];
void rotate(int num, int dir);

int main() {
	for (int i = 0; i < 4; ++i) {
		scanf("%s", wheel[i]);
	}
	scanf("%d", &K);

	while (K--) {
		int num, dir;
		scanf("%d %d", &num, &dir);
		--num;
		memset(rotated, false, sizeof(rotated));
		rotate(num, dir);
	}

	int score = 0;
	if (wheel[0][0] == '1') score += 1;
	if (wheel[1][0] == '1') score += 2;
	if (wheel[2][0] == '1') score += 4;
	if (wheel[3][0] == '1') score += 8;
	printf("%d", score);
	return 0;
}

void rotate(int num, int dir) {
	rotated[num] = true;
	char tmp;
	int l = num - 1, r = num + 1;

	if (0 <= l && wheel[l][2] != wheel[num][6] && !rotated[l]) {
		rotate(l, dir * -1);
	}
	if (r < 4 && wheel[num][2] != wheel[r][6] && !rotated[r]) {
		rotate(r, dir * -1);
	}

	if (dir == 1) {
		tmp = wheel[num][7];
		for (int i = 6; i >= 0; --i) {
			wheel[num][i + 1] = wheel[num][i];
		}
		wheel[num][0] = tmp;
	}
	else {
		tmp = wheel[num][0];
		for (int i = 0; i <= 6; ++i) {
			wheel[num][i] = wheel[num][i + 1];
		}
		wheel[num][7] = tmp;
	}
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
```
</details>

### 설명
1. 좌 / 우 또는 상 / 하 의 범위를 확인해야 하는 문제는 인덱스를 1부터 저장하면, 범위를 넘어가는 예외를 간단히 처리할 수 있다.
   
   이 문제에서는 hori 배열의 인덱스를 1부터 저장함으로써, 좌 / 우를 확인할 때 범위를 확인하지 않도록 할 수 있다. 

2. 2차원 배열 위에서 dfs를 할 때, (y, x) 값부터 탐색을 하도록 행과 열 값을 입력받도록 하면 순서만 다를 뿐 같은 경우의 수를 중복해서 세지 않을 수 있다.

   NxM board 위에서 dfs를 수행한다고 가정하자.
   
   dfs(y, x) 내부에서 board[y][x ~ M - 1] 탐색 이후, board[y + 1 ~ N - 1][0 ~ M - 1] 을 탐색하도록 구현할 수 있다.

***

## 15685: 드래곤 커브
> https://www.acmicpc.net/problem/15685

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
using namespace std;

const int dy[4] = { 0, -1, 0, 1 }, dx[4] = { 1, 0, -1, 0 };
int N, ret;
bool exist[101][101];
vector<int> dragon;

void make_dragon(int y, int x, int g);
void count_square();

int main() {
	scanf("%d", &N);
	while (N--) {
		int x, y, d, g;
		scanf("%d %d %d %d", &x, &y, &d, &g);

		dragon.clear();
		
		dragon.push_back(d);
		exist[y][x] = exist[y + dy[d]][x + dx[d]] = true;
		make_dragon(y + dy[d], x + dx[d], g);
	}
	count_square();
	printf("%d", ret);
	return 0;
}

void count_square() {
	for (int y = 0; y <= 99; ++y) {
		for (int x = 0; x <= 99; ++x) {
			if (exist[y][x] && exist[y][x + 1] && exist[y + 1][x] && exist[y + 1][x + 1]) {
				++ret;
			}
		}
	}
}

void make_dragon(int y, int x, int g) {
	if (g == 0) return;
	for (int i = dragon.size() - 1; i >= 0; --i) {
		int d = (dragon[i] + 2) % 4;
		d = (d + 3) % 4;
		y += dy[d]; x += dx[d];
		exist[y][x] = true;
		dragon.push_back(d);
	}
	make_dragon(y, x, g - 1);
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

***

## 5373. 큐빙
> https://www.acmicpc.net/problem/5373

### 코드
<details>
<summary>C++</summary>

```cpp
#include <cstdio>
#include <cstring>

enum Face { U, D, F, R, B, L };
struct Pos {
	int f, y, x;
};

char dice[6][3][3], tmp[6][3][3];
Pos to[6][2][6][3][3]; // to[f2][d][f][y][x] : f면 y, x 를 f2면 d방향 회전시 이동 좌표

void init();
void reset_dice();

void rotate(char f, char d);

int main() {
	init();
	int T, n;
	char f, d;
	scanf("%d", &T);
	while (T--) {
		scanf("%d", &n);
		getchar();
		reset_dice();
		while (n--) {
			scanf("%c%c", &f, &d);
			getchar();
			rotate(f, d);
		}
		for (int y = 0; y < 3; ++y) {
			for (int x = 0; x < 3; ++x) {
				printf("%c", dice[U][y][x]);
			}
			printf("\n");
		}
	}
	return 0;
}

void rotate(char f, char d) {
	int face;
	if (f == 'U') face = 0;
	else if (f == 'D') face = 1;
	else if (f == 'F') face = 2;
	else if (f == 'R') face = 3;
	else if (f == 'B') face = 4;
	else if (f == 'L') face = 5;

	int dir = (d == '+' ? 0 : 1);

	for (int f = 0; f < 6; ++f) {
		for (int y = 0; y < 3; ++y) {
			for (int x = 0; x < 3; ++x) {
				Pos& p = to[face][dir][f][y][x];
				tmp[p.f][p.y][p.x] = dice[f][y][x];
			}
		}
	}

	memcpy(dice, tmp, sizeof(dice));
}

void reset_dice() {
	for (int f = 0; f < 6; ++f) {
		memset(dice[f], "wyrbog"[f], 9);
	}
}

void init() {
	// 0. 먼저 모두 자기 자신으로 가도록
	for (int f = 0; f < 6; ++f) {
		for (int dir = 0; dir < 2; ++dir) {
			for (int ff = 0; ff < 6; ++ff) {
				for (int y = 0; y < 3; ++y) {
					for (int x = 0; x < 3; ++x) {
						to[f][dir][ff][y][x] = { ff, y, x };
					}
				}
			}
		}
	}
	// 1. U면
	for (int i = 0; i < 3; ++i) {
		to[U][0][F][0][i] = { L, 0, i };
		to[U][1][F][0][i] = { R, 0, i };
	}
	for (int i = 0; i < 3; ++i) {
		to[U][0][L][0][i] = { B, 0, i };
		to[U][1][L][0][i] = { F, 0, i };
	}
	for (int i = 0; i < 3; ++i) {
		to[U][0][B][0][i] = { R, 0, i };
		to[U][1][B][0][i] = { L, 0, i };
	}
	for (int i = 0; i < 3; ++i) {
		to[U][0][R][0][i] = { F, 0, i };
		to[U][1][R][0][i] = { B, 0, i };
	}
	for (int y = 0; y < 3; ++y) {
		for (int x = 0; x < 3; ++x) {
			to[U][0][U][y][x] = { U, x, 2 - y };
			to[U][1][U][y][x] = { U, 2 - x, y };
		}
	}
	// 2. D면 (위에서 볼 땐 반대로 회전)
	for (int i = 0; i < 3; ++i) {
		to[D][1][F][2][i] = { L, 2, i };
		to[D][0][F][2][i] = { R, 2, i };
	}
	for (int i = 0; i < 3; ++i) {
		to[D][1][L][2][i] = { B, 2, i };
		to[D][0][L][2][i] = { F, 2, i };
	}
	for (int i = 0; i < 3; ++i) {
		to[D][1][B][2][i] = { R, 2, i };
		to[D][0][B][2][i] = { L, 2, i };
	}
	for (int i = 0; i < 3; ++i) {
		to[D][1][R][2][i] = { F, 2, i };
		to[D][0][R][2][i] = { B, 2, i };
	}
	for (int y = 0; y < 3; ++y) {
		for (int x = 0; x < 3; ++x) {
			to[D][1][D][y][x] = { D, x, 2 - y };
			to[D][0][D][y][x] = { D, 2 - x, y };
		}
	}
	// 3. F면
	for (int i = 0; i < 3; ++i) {
		to[F][0][U][2][i] = {R, i, 0};
		to[F][1][U][2][i] = {L, 2 - i, 2};
	}
	for (int i = 0; i < 3; ++i) {
		to[F][0][R][i][0] = {D, 2, 2 - i};
		to[F][1][R][i][0] = {U, 2, i};
	}
	for (int i = 0; i < 3; ++i) {
		to[F][0][D][2][i] = {L, i, 2};
		to[F][1][D][2][i] = {R, 2 - i, 0};
	}
	for (int i = 0; i < 3; ++i) {
		to[F][0][L][i][2] = {U, 2, 2 - i};
		to[F][1][L][i][2] = {D, 2, i};
	}
	for (int y = 0; y < 3; ++y) {
		for (int x = 0; x < 3; ++x) {
			to[F][0][F][y][x] = { F, x, 2 - y };
			to[F][1][F][y][x] = { F, 2 - x, y };
		}
	}
	// 4. R면
	for (int i = 0; i < 3; ++i) {
		to[R][0][U][i][2] = {B, 2 - i, 0};
		to[R][1][U][i][2] = {F, i, 2};
	}
	for (int i = 0; i < 3; ++i) {
		to[R][0][B][i][0] = {D, i, 2};
		to[R][1][B][i][0] = {U, 2 - i, 2};
	}
	for (int i = 0; i < 3; ++i) {
		to[R][0][D][i][2] = {F, 2 - i, 2};
		to[R][1][D][i][2] = {B, i, 0};
	}
	for (int i = 0; i < 3; ++i) {
		to[R][0][F][i][2] = {U, i, 2};
		to[R][1][F][i][2] = {D, 2 - i, 2};
	}
	for (int y = 0; y < 3; ++y) {
		for (int x = 0; x < 3; ++x) {
			to[R][0][R][y][x] = { R, x, 2 - y };
			to[R][1][R][y][x] = { R, 2 - x, y };
		}
	}
	// 4. B면
	for (int i = 0; i < 3; ++i) {
		to[B][0][U][0][i] = {L, 2 - i, 0};
		to[B][1][U][0][i] = {R, i, 2};
	}
	for (int i = 0; i < 3; ++i) {
		to[B][0][L][i][0] = {D, 0, i};
		to[B][1][L][i][0] = {U, 0, 2 - i};
	}
	for (int i = 0; i < 3; ++i) {
		to[B][0][D][0][i] = {R, 2 - i, 2};
		to[B][1][D][0][i] = {L, i, 0};
	}
	for (int i = 0; i < 3; ++i) {
		to[B][0][R][i][2] = {U, 0, i};
		to[B][1][R][i][2] = {D, 0 , 2 - i};
	}
	for (int y = 0; y < 3; ++y) {
		for (int x = 0; x < 3; ++x) {
			to[B][0][B][y][x] = { B, x, 2 - y };
			to[B][1][B][y][x] = { B, 2 - x, y };
		}
	}
	// 3. L면
	for (int i = 0; i < 3; ++i) {
		to[L][0][U][i][0] = {F, i, 0};
		to[L][1][U][i][0] = {B, 2 - i, 2};
	}
	for (int i = 0; i < 3; ++i) {
		to[L][0][F][i][0] = {D, 2 - i, 0};
		to[L][1][F][i][0] = {U, i, 0};
	}
	for (int i = 0; i < 3; ++i) {
		to[L][0][D][i][0] = {B, i, 2};
		to[L][1][D][i][0] = {F, 2 - i, 0};
	}
	for (int i = 0; i < 3; ++i) {
		to[L][0][B][i][2] = {U, 2 - i, 0};
		to[L][1][B][i][2] = {D, i, 0};
	}
	for (int y = 0; y < 3; ++y) {
		for (int x = 0; x < 3; ++x) {
			to[L][0][L][y][x] = { L, x, 2 - y };
			to[L][1][L][y][x] = { L, 2 - x, y };
		}
	}
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
#include <cstdio>
#include <cstring>
#include <queue>
#include <vector>
using namespace std;

struct Pos {
	int y, x;
};

const int dy[4] = { -1, 1, 0, 0 }, dx[4] = { 0, 0, -1, 1 };
int N, L, R, board[50][50];
void input();

bool discovered[50][50];
bool move();

int main() {
	input();

	for(int day = 0; ; ++day) {
		if (!move()) {
			printf("%d", day);
			break;
		}
	}

	return 0;
}

bool can_move(Pos a, Pos b) {
	int gap = board[a.y][a.x] - board[b.y][b.x];
	if (gap < 0) gap *= -1;
	return L <= gap && gap <= R;
}

int bfs(int y, int x, vector<Pos>& v) {
	queue<Pos> q;
	q.push({ y, x });
	discovered[y][x] = true;

	int sum = 0;

	while (!q.empty()) {
		Pos here = q.front(); q.pop();
		sum += board[here.y][here.x];
		v.push_back(here);
		for (int d = 0; d < 4; ++d) {
			int yy = here.y + dy[d], xx = here.x + dx[d];
			if (yy < 0 || yy >= N || xx < 0 || xx >= N) continue;
			if(discovered[yy][xx] || !can_move(here, {yy, xx})) continue;
			q.push({ yy, xx });
			discovered[yy][xx] = true;
		}
	}

	return sum;
}

bool move() {
	memset(discovered, false, sizeof(discovered));
	bool check = false;

	for (int y = 0; y < N; ++y) {
		for (int x = 0; x < N; ++x) {
			if (discovered[y][x]) continue;
			vector<Pos> v;
			int sum = bfs(y, x, v);
			if (v.size() > 1) {
				check = true;
				sum /= v.size();
				for (Pos& p : v) {
					board[p.y][p.x] = sum;
				}
			}
		}
	}

	return check;
}

void input() {
	scanf("%d %d %d", &N, &L, &R);
	for (int i = 0; i < N; ++i) {
		for (int j = 0; j < N; ++j) {
			scanf("%d", &board[i][j]);
		}
	}
}
```
</details>

### 설명
bfs 문제.

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

## 문제
> 링크

### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

### 설명

***

## 문제
> 링크

### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

### 설명

***

## 문제
> 링크

### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

### 설명

***

## 문제
> 링크

### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

### 설명

***

## 문제
> 링크

### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

### 설명

***

## 문제
> 링크

### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

### 설명

***