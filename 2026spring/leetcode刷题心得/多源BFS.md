[（多源BFS）994. 腐烂的橘子 - 力扣（LeetCode）](https://leetcode.cn/problems/rotting-oranges/submissions/695022078/?envType=study-plan-v2&envId=top-100-liked)
[（正常BFS）200. 岛屿数量 - 力扣（LeetCode）](https://leetcode.cn/problems/number-of-islands/description/?envType=study-plan-v2&envId=top-100-liked)

#### 1. 思维模型的转变：从“多线并发”到“洪水泛滥”

- **我的旧误区**：把每个腐烂的橘子看作独立的个体，每个人维护一个队列（`vector<queue>`），轮流走一步。感觉像是在模拟几个不同的“特工”分头行动。
    - _缺点_：逻辑极度复杂，维护成本高，且无法统一“时间”的概念。
- 具体代码如下：
```cpp
class Solution {
public:
    bool check(int x, int y, int m, int n, vector<vector<int>>& grid){
        return x < m && y < n && x >= 0 && y >= 0 && grid[x][y] != 0;
    }
	
    vector<queue<pair<pair<int, int>, int>>> q;
    bool empty(){
        int flag = 1;
        for(int i=0; i<q.size(); i++)
            if(!q[i].empty())
                flag = 0;
        return flag;
    }
	
    int bfs(int m, int n, vector<vector<int>>& grid){
        int dir[4][2] = {
            {0, 1}, {0, -1}, {1, 0}, {-1, 0}
        };
        int step = 0;
	
        while(!empty()){
            for(int k=0; k<q.size(); k++){
                if(q[k].empty()) continue;
                //错在这个循环里面！！ 处理了多个根节点，但是每个st却只处理了一个子节点
		        /*这里可以补救，但是非常冗余：
				int cur = q[k].size();
                for(int l=0; l<cur; l++){
                for(int i=0; i<=3; i++){
                    int nx = q[k].front().first.first + dir[i][0];
                    int ny = q[k].front().first.second + dir[i][1];
                    int st = q[k].front().second;
                    if(check(nx, ny, m, n, grid)){
                        q[k].push({{nx, ny}, st+1});
                        grid[nx][ny] = 0;
                        step = max(step, st+1);
                    }
                }
                q[k].pop();
                }*/
                
                for(int i=0; i<=3; i++){
                    int nx = q[k].front().first.first + dir[i][0];
                    int ny = q[k].front().first.second + dir[i][1];
                    int st = q[k].front().second;
                    if(check(nx, ny, m, n, grid)){
                        q[k].push({{nx, ny}, st+1});
                        grid[nx][ny] = 0;
                        step = max(step, st+1);
                    }
                }
                q[k].pop();
            }
        }
        return step;
    }
    
    int orangesRotting(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size(), step;
        for(int i=0; i<m; i++){
            for(int j=0; j<n; j++){
                if(grid[i][j] == 2){
                    grid[i][j] = 0;
                    q.emplace_back().push({{i, j}, 0});
                }
            }
        }

        step = bfs(m, n, grid);
        for(int i=0; i<m; i++){
            for(int j=0; j<n; j++){
                //cout <<grid[i][j]<<" ";
                if(grid[i][j])
                    return -1;
            }
            cout <<endl;
        }
        return step;
    }
};
```


- **正确的模型**：**洪水（Flood）模型**。
    - 不需要区分“这个病毒是谁传过来的”。所有烂橘子在第 0 分钟都是“源头”。
    - **核心心法**：**把所有源头扔进同一个队列**。一旦入队，它们就失去了个体身份，只代表“当前这一层”的波浪。

#### 2. 代码结构的灵魂：`cnt = q.size()` 的决定性作用

- **最痛的教训**：BFS 如果没有内层的 `for(int i=0; i < size; i++)` 循环，它就只是一个“遍历工具”，而失去了“分层（计时）功能”。

- **差异对比**：
    - **没有 `size` 循环**：队列是流动的，你处理完第 1 个节点，刚塞进去的邻居（第 2 层）马上就排到了第 3 个位置。你根本不知道哪一个节点属于第几分钟。
    - **有 `size` 循环**：这是一个**时间快照**。意味着：“请先处理完当前这一分钟内的**所有**任务，处理完了，时间才能 +1”。
	
    - **结论**：**求最短路、求层数、求分钟数，必须加 `size` 循环！**


#### 3. 状态管理的优化：动态计数 vs 事后诸葛

- **旧做法**：BFS 跑完后，再写两个 `for` 循环遍历整个二维数组去查有没有剩下的新鲜橘子。
    - _代价_：$O(M \times N)$ 的额外时间开销。
        
- **最优解**：**动态计数 (On-the-fly counting)**。
    - 初始化时统计 `fresh_cnt`。
    - 每感染一个，`fresh_cnt--`。
    - 结束时直接看 `fresh_cnt == 0`。
    - _心得_：能在过程里做完的事，绝不留到最后再遍历。

---

### 🚀 你的多源 BFS 黄金模板

以后遇到任何“病毒扩散”、“同时起火”、“多点最短路”问题，直接套用这个结构，一行代码都不用多写：

```cpp
// 1. 定义方向：右左下上
int dir[4][2] = {{0, 1}, {0, -1}, {1, 0}, {-1, 0}};

int bfs(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    queue<pair<int, int>> q;
    int target_count = 0; // 这里的 target 可以是新鲜橘子，也可以是待填充的区域

    // --- 第一步：全员入队 (初始化第 0 层) ---
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (grid[i][j] == SOURCE) { // 比如烂橘子
                q.push({i, j});
            } else if (grid[i][j] == TARGET) { // 比如新鲜橘子
                target_count++;
            }
        }
    }

    // 特判：如果一开始就没目标，直接返回
    if (target_count == 0) return 0;

    int step = 0; // 记录层数/时间

    // --- 第二步：按层扩散 ---
    while (!q.empty()) {
        int size = q.size(); // 【核心】：锁定当前层节点数
        bool has_spread = false; // 标记这一层有没有实际扩散

        for (int k = 0; k < size; ++k) { // 处理完这一层的每一个节点
            auto [x, y] = q.front(); 
            q.pop();

            for (int d = 0; d < 4; ++d) {
                int nx = x + dir[d][0];
                int ny = y + dir[d][1];

                // 越界检查 + 有效性检查
                if (nx >= 0 && nx < m && ny >= 0 && ny < n && grid[nx][ny] == TARGET) {
                    grid[nx][ny] = SOURCE; // 直接修改状态，避免 visited 数组
                    q.push({nx, ny});
                    target_count--;
                    has_spread = true;
                }
            }
        }
        // 这一层全部处理完，时间才走一格
        if (has_spread) step++; 
    }

    // --- 第三步：结果校验 ---
    return target_count == 0 ? step : -1;
}
```