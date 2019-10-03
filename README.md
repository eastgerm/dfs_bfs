# 개천절에 연구소 파헤치기

## 필요 스킬
### 2차원 배열 요소 탐색
```
- for문 (무식하게)
- DFS (깊이 우선 탐색)
- BFS (너비 우선 탐색)
```
- 어떤 방법이 가장 효과적일까 ?

- DFS와 BFS는 어느 부분이 다른가?

- 각각 어떤 유형의 문제에서 장점이 있을까?
    ```
    - 미로 탈출 최소 경로 찾기 -> DFS? BFS?
    - 사전 순으로 출력하기 -> DFS? BFS?
    ```

<br/>

### 조합 뽑아내기 
```
- for문 (무식하게)
- DFS or BFS (직접 조합의 경우의 수 생성)
- next_permultation (STL에서 제공하는 조합의 경우의 수 사용)
```
- 세워야하는 벽이 13개여도 for문을 선택할 것 인가?

<br/>

## 문제풀이 시나리오

1. 입력

2. 벽 세울 3곳 선택

3. 벽 설치

4. 바이러스 확산

5. 안전영역 카운트 `(최대값이라면 값 갱신)`

6. map 초기상태 복구 `(왜 해야할까?)`

7. `선택 할 수 있는 모든 조합으로 벽 설치를 완료` ? `종료` : `2번으로 돌아가기`

8. 맞았습니다


<br/>

## eastgerm의 코드
```
#include <iostream>
#include <vector>
#include <algorithm>
//bfs
#include <queue>

using namespace std;

#define max(x, y) ((x)>(y)?(x):(y))

struct vertex {
    int row;
    int col;
};
int dR[4] = {-1, 0, 1, 0};
int dC[4] = {0, 1, 0, -1};

int N, M;
vector<vector<int>> map, tmap;
vector<vector<bool>> visit;
vector<vertex> virus, room;
vector<int> pickedRoom;
int virusLength;
int roomLength;
int nowRoomLength;
//bfs
queue<vertex> bomb;

bool isOutOfRange(vertex ver) {
    return ver.row < 0 || ver.row >= N || ver.col < 0 || ver.col >= M;
}

bool isNotRoom(vertex ver) {
    return tmap[ver.row][ver.col] != 0;
}

bool isVisited(vertex ver) {
    return visit[ver.row][ver.col];
}

bool isMaintainRoom(int idx) {
    return pickedRoom[idx] == 1;
}

void input();

void pickFirstThreeRooms();

void goDfs(vertex ver);

//bfs
void pushQ(vertex ver);
void goBfs(vertex ver);
void popQ();

void init();

void makeNewWall();

void spreadVirusDfs();

//bfs
void spreadVirusBfs();

int safeZone();

void output();

int main() {
    ios::sync_with_stdio(false), cin.tie(NULL);
    input();
    output();
    return 0;
}

void input() {
    cin >> N >> M;
    map.assign(N, vector<int>(M, 0));
    for (int i = 0; i < N; i++)
        for (int j = 0; j < M; j++) {
            cin >> map[i][j];
            if (map[i][j] == 0) {
                room.push_back({i, j});
                pickedRoom.push_back(1);
                roomLength++;
            } else if (map[i][j] == 2) {
                virus.push_back({i, j});
                virusLength++;
            }
        }
    pickFirstThreeRooms();
}

void pickFirstThreeRooms() {
    pickedRoom[0] = pickedRoom[1] = pickedRoom[2] = 0;
}

void goDfs(vertex ver) {
    visit[ver.row][ver.col] = true;
    tmap[ver.row][ver.col] = 2;
    for (int i = 0; i < 4; i++) {
        vertex nowVer = {ver.row + dR[i], ver.col + dC[i]};
        if (isOutOfRange(nowVer)) continue;
        if (isNotRoom(nowVer)) continue;
        if (isVisited(nowVer)) continue;
        nowRoomLength--;
        goDfs(nowVer);
    }
}

//bfs
void pushQ(vertex ver) {
    bomb.push(ver);
    visit[ver.row][ver.col] = true;
}
void goBfs(vertex ver) {
    tmap[ver.row][ver.col] = 2;
    for (int i = 0; i < 4; i++) {
        vertex nowVer = {ver.row + dR[i], ver.col + dC[i]};
        if (isOutOfRange(nowVer)) continue;
        if (isNotRoom(nowVer)) continue;
        if (isVisited(nowVer)) continue;
        nowRoomLength--;
        pushQ(nowVer);
    }
}
void popQ() {
    while(!bomb.empty()) {
        vertex nowVer = bomb.front();
        goBfs(nowVer);
        bomb.pop();
    }
}

void init() {
    tmap = map;
    visit.assign(N, vector<bool>(M, false));
    nowRoomLength = roomLength;
}

void makeNewWall() {
    int cnt = 0;
    for (int i = 0; i < roomLength; i++) {
        if (cnt == 3) break;
        if (isMaintainRoom(i)) continue;
        vertex nowVer = {room[i].row, room[i].col};
        tmap[nowVer.row][nowVer.col] = 1;
        nowRoomLength--;
        cnt++;
    }
}

void spreadVirusDfs() {
    for (int i = 0; i < virusLength; i++) {
        vertex nowVer = {virus[i].row, virus[i].col};
        goDfs(nowVer);
    }
}

//bfs
void spreadVirusBfs() {
    for (int i = 0; i < virusLength; i++) {
        vertex nowVer = {virus[i].row, virus[i].col};
        pushQ(nowVer);
    }
    popQ();
}

int safeZone() {
    int ans = 0;
    do {
        init();
        makeNewWall();
        //spreadVirusDfs();
        spreadVirusBfs();
        ans = max(ans, nowRoomLength);
    } while (next_permutation(pickedRoom.begin(), pickedRoom.end()));
    return ans;
}

void output() {
    cout << safeZone() << '\n';
}
```
