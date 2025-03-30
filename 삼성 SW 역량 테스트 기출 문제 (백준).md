# 삼성 SW 역량 테스트 기출 문제 (백준)
## 13460. 구슬 탈출 2
> https://www.acmicpc.net/problem/13460
***
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

***
### 설명
1. red와 blue 값을 매개변수로 넘기지 않고, 보드를 기울이기 전에 red와 blue 값을 저장해두면 전역변수로 사용할 수 있다. 

   => dfs 함수에 매개변수로 넘기지 않고 깔끔하게 작성 가능하다.
2. 보드를 같은 방향으로 연속해서 두 번 이상 기울이는건 의미 없다. 

   => 가지치기.
***

## 12100. 2048 (Easy)
> https://www.acmicpc.net/problem/12100
***
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

***
### 설명
1. 가장 큰 블록만을 출력하면 되는 문제이므로, 보드를 회전하여도 상관 없다. 따라서 보드를 회전시키며 블록을 이동하면, 한 쪽 방향으로 이동하는 부분만 구현하면 된다.
2. Info 값을 매개변수로 넘기는게 아니라, 전역변수로 설정하고 재귀호출 전에 미리 저장해놓는 방식으로 구현하면 코드가 훨씬 깔끔해진다.
3. 보드를 Info struct 에 집어넣고, 보드의 회전, 블록의 이동, 가장 큰 블록 구하기 연산을 구조체 함수로 구현하면 코드가 깔끔하다.
***

## 3190. 뱀
> https://www.acmicpc.net/problem/3190
***
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

***
### 설명
deque 활용 문제
***

## 13458: 시험 감독
> https://www.acmicpc.net/problem/13458
***
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

***
### 설명
1. int형 범위를 벗어날 가능성이 있다면 long long 자료형을 사용하기.
2. A를 B로 나눈 후 올림한 값 => (A + B - 1) / B;
***

## 14499. 주사위 굴리기 
> https://www.acmicpc.net/problem/14499
***
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

***
### 설명
1. 주석을 잘 활용하자.
2. char 변수는 scanf에서는 %hhd, printf에서는 %d 사용한다.
3. unsigned char 변수는 scanf 에서는 %hhu, printf에서는 %u 사용한다.
4. unsigned int 변수는 scanf 에서는 %u, printf에서는 %u 사용한다.
***

## 14500: 테트로미노
> https://www.acmicpc.net/problem/14500
***
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

***
### 설명
vector<T> 대상으로 emplace_back() 연산을 하면, T() 가 vector의 맨 뒤로 삽입된다. 이를 이용하여 필요할 때마다 vector의 크기를 늘릴 수 있다.
***

## 문제
> 링크
***
### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

***
### 설명

***

## 문제
> 링크
***
### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

***
### 설명

***

## 문제
> 링크
***
### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

***
### 설명

***

## 문제
> 링크
***
### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

***
### 설명

***

## 문제
> 링크
***
### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

***
### 설명

***

## 문제
> 링크
***
### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

***
### 설명

***

## 문제
> 링크
***
### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

***
### 설명

***

## 문제
> 링크
***
### 코드
<details>
<summary>C++</summary>

```cpp
```
</details>

***
### 설명

***
