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
