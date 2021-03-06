# 트리에서의 최장 경로

## $O(N^2)$ 알고리즘

각 정점에 대해서 BFS(또는 DFS)를 해서, 가장 긴 경로 찾기.

## $O(N)$ 알고리즘

1. 임의의 정점에서 시작. 탐색해서 가장 멀리 떨어진 정점 $V$ 찾기.
2. $V$에서 찾은 최장 경로가 구하는 경로.

### 증명

1단계에서 찾은 정점이 최장 경로의 끝점중 하나임을 보이면 된다.

처음 탐색을 시작한 정점을 $v_0$라고 하고, 1단계에서 찾은 정점을 $v$라고 둔 다음, $v$가 최장 경로에 포함되지 않는다고 가정하자.

최장 경로가 $u$에서 $w$로 가는 경로라 하고, 가장 $v_0$에 가까운 점을 $h$라 하자. 이때 두 가지 경우를 생각해 볼 수 있다.

1. $v_0-v$ 경로에 $h$가 포함되는 경우.
   - $h-u$, $h-w$ 거리 둘 다 $h-v$ 거리보다 긴 것은 불가능하다.
   - 이때 $h-u$, $h-w$ 경로 둘 중 짧은 것을 $h-v$ 경로로 바꾸면 가정한 최장 경로보다 긴 경로를 얻는다. 모순.
2. $v_0-v$ 경로에 $h$가 포함되지 않는 경우.
   - $v-h$ 경로에서 $v_0$에 가장 가까운 점을 $h'$이라 하자.
   - $h'-u$, $h'-w$ 거리 둘 다 $h'-v$ 거리보다 긴 것은 불가능하다.
   - $h'-w$ 경로는 $h-w$ 경로보다, $h'-v$ 경로는 $h-v$ 경로보다 길다.
   - $v-u$, $v-w$ 경로 모두 $h'$을 포함한다. 일반성을 잃지 않고 $v-u$가 더 길다고 하자.
   - $v-u$ 경로의 길이는 $h'-v$, $h'-u$ 경로 길이의 합이다.
   - $h'-v$ 경로는 $h'-w$ 경로보다 짧을 수 없고, $h'-w$ 경로는 $h-w$ 경로보다 길다.
   - $h'-u$ 경로는 $h-u$ 경로보다 길다.
   - $u-w$ 경로의 길이는 $h-u$, $h-w$ 경로 길이의 합이다.
   - 가정한 최장 경로보다 $v-u$ 경로의 길이가 더 길다. 모순.

