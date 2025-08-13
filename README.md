A sokoban game solver
===========================
This project proposed a AI solver for sokoban (japanese for warehouse keeper) which is a difficult computational problem. The algorithm being used consisted of BFS (breadth first search), DFS (depth first search), UCS (uniform cost search) and A* (A star search).

## 目錄

* [0. 使用方法](#0)
* [1. 總覽](#1)
* [2. 結果對比](#2)

<a id="0"></a>
## 0. 使用方法

1. 需要import的有：`sys`，`collections`，`numpy`，`heapq`，`time`。

2. 下載到本機後運行`sokoban.py`文件即可。

### 查看幫助

```
$ python sokoban.py --help
```
```
Usage: sokoban.py [options]

Options:
  -h, --help            show this help message and exit
  -l SOKOBANLEVELS, --level=SOKOBANLEVELS
                        level of game to play (test1-10.txt, level1-5.txt)
  -m AGENTMETHOD, --method=AGENTMETHOD
                        research method (bfs, dfs, ucs, astar)
```

`-l`：地圖分為test和level，test的比較簡單，level的比較難。
`-m`：搜尋算法為bfs，dfs，ucs或astar。

### 運行示例

```
$ python sokoban.py -l test1.txt -m bfs
```
```
rUUdRdrUUluL
Runtime of bfs: 0.15 second.
```
第一行輸出為推箱者的動作，`u`，`d`，`l`，`r`分別代表上、下、左、右的移動，對應的大寫字母則代表該方向上推著箱子移動。第二行輸出為程式的運行時間。

<a id="1"></a>
## 1. 總覽

Sokoban，也就是推箱子遊戲，玩家需要把所有箱子推到目的地才算成功。

### 地圖示例（`level1.txt`）

**圖片形式：**

![](./img/level1.png)

**輸入形式：**

```
  #####
###   #
#.&B  #
### B.#
#.##B #
# # . ##
#B XBB.#
#   .  #
######## 
```

其中`#`是牆，`.`是目的地，`B`是箱子（`X`是箱子在目的地上），`&`是推箱子的人（`%`是箱子在目的地上），空白處為可移動空間。

### 思路

搜索算法在這裡的使用，簡單理解就是當前狀態下推箱者採取一個可能動作後，產生下一狀態，以此類推，直到最終狀態為結束狀態。既然用到的是搜索算法，首先要確定的是state space graph（SSG）和search tree（ST）。如果直接把輸入的整個地圖作為SSG存儲在數據結構中，會使內存的佔用隨搜索的進行而飆升，所以必須重新定義新的SSG，既能減少空間佔用，又能代表足夠的信息。在推箱子這個問題中，其實關鍵的部分僅為箱子的位置和人的位置，因為牆和目的地的位置都是不變的。所以定義SSG的樣式如下所示，其中第一個tuple代表當前人的坐標，第二個tuple代表當前箱子的坐標。

((2, 2), ((2, 3), (3, 4), (4, 4), (6, 1), (6, 4), (6, 5)))

而ST就是當前SSG基於推箱者可行的動作進行分枝，每一個動作都會導致新的不同的SSG產生，最終形成很多個branch和node。只要所有箱子的坐標和目的地的坐標完全契合，就說明遊戲結束（勝利）。但推箱子遊戲，往往會把箱子推到一些位置，比如說死角，這種局面其實也會導致遊戲結束（失敗），就沒有繼續走下去的必要了。因此，利用這些導致死局的pattern可以幫助我們修剪樹（prune tree），使分裂的node減少許多，進而減少內存的佔用。下圖展示一些死局的pattern，以一個箱子為中心，如果周圍一圈內出現這些情況，則說明該當前態沒有再分裂下去的必要了。

![](./img/dead_patterns.jpg)

另外為了避免推箱者做無意義的移動，比如不推箱子來回晃悠，我們要讓每次分枝後的SSG與同一條分枝上的SSG不重復，即某個“人的位置和所有箱子的位置”不能重復出現。

然後就是算法的部分，BFS和DFS就不多說了，一個“每條路都走一小步慢慢走”，一個”一條路走到死再走下一條“。而UCS加入了cost function，就是”挑最節省成本的路走“，這裡的cost function定義為：到目當前狀態為止，所有沒推著箱子走的步數。這樣能夠激勵推箱者盡可能做有意義的移動，而不是不推箱子逛該。最後A*，在cost function上加入heuristic function，目的是激勵推箱者不單去推箱子，並且還要把箱子推往目的地。這裡的heuristic function用的是manhatten distance，定義為：到目當前狀態為止，所有箱子按順序排列的位置和所有目的地按順序排列的位置，兩兩間的manhatten distance總和。

<a id="2"></a>
## 2. 結果對比

* BFS：

```
$ python sokoban.py -l test1.txt -m bfs
rUUdRdrUUluL
Runtime of bfs: 0.15 second.

$ python sokoban.py -l test1.txt -m bfs
rUUdRdrUUluL
Runtime of bfs: 0.13 second.

$ python sokoban.py -l test2.txt -m bfs
UUddrrrUU
Runtime of bfs: 0.01 second.

$ python sokoban.py -l test3.txt -m bfs
LrdrddDLdllUUdR
Runtime of bfs: 0.27 second.

$ python sokoban.py -l test4.txt -m bfs
llldRRR
Runtime of bfs: 0.01 second.

test5.txt: more than 1 minute.

$ python sokoban.py -l test6.txt -m bfs
dlluRdrUUUddrruuulL
Runtime of bfs: 0.02 second.

$ python sokoban.py -l test7.txt -m bfs
LUUUluRddddLdlUUUUluR
Runtime of bfs: 1.25 second.

$ python sokoban.py -l test8.txt -m bfs
llDDDDDDldddrruuLuuuuuuurrdLulDDDDDDlllddrrUdlluurRdddrruuLUUUUUUluRdddddddrddlluUdlluurRdrUUUUUU
Runtime of bfs: 0.30 second.

$ python sokoban.py -l level1.txt -m bfs
RurrddddlDRuuuuLLLrdRDrddlLdllUUdR
Runtime of bfs: 35.04 second.
```

* DFS：

```
$ python sokoban.py -l test1.txt -m dfs
rrUrdllluRRlldrrrUlllururrDLrullldRldrrUruLrdllldrrdrUlllurrurDluLrrdllldrrdrUU
Runtime of dfs: 0.09 second.

$ python sokoban.py -l test2.txt -m dfs
rrrUlldlUrrrUlldrrdllluU
Runtime of dfs: 0.01 second.

$ python sokoban.py -l test3.txt -m dfs
rrdldrdllDRlldlUrrrdrruLrdllLrrrululldRllldRlurrrdrruLrdllUrrdllLrrrullllldRRRlllurrrrrdLrulllllUdrrrrrdlLrrullllldrRRlllurrrrrdLrullluRldrrrdlLrrullllldrRRlllurrrrrdLrulllururulluLrrrdlddldrrrdlLrrullllldrRRlllurrrrrdLrulUlldrrrdlLrrullllldrRRlllurrrrrdLrulllurrUldrdrdlLrrullllldrRRlllurrrrrdLrulllurrululurrDDldldrrrdlLrrulllururDlldrdrruLrdllLrrrululldRllldRlurrrdLrrruLrdlllulldRRRlllurrurrdrdLrulL
Runtime of dfs: 0.36 second.

$ python sokoban.py -l test4.txt -m dfs
rrrdllllLrrrrrdllllllluRRRR
Runtime of dfs: 0.01 second.

test5.txt: more than 1 minute.

$ python sokoban.py -l test6.txt -m dfs
rrdlllluRRlldrrrruLrdllUUUddrrdlllluuuurRlldddrrrruuuLL
Runtime of dfs: 0.02 second.

$ python sokoban.py -l test7.txt -m dfs
rdllllurRlldrrrruulLrrdLrdllllurRlldrrrruullLrrrdLrdllllurRlldrrrruulluulldDrrrrdLrdllllUrdrrrulLrrdlllluUrrrrdllLrrrullllUdrrrrdllldlUrrrrulluululldRRllurrrrrdddllldrrrdlllluUrrrruuullllldrDDrrrrdllldlUrrrrulluUlllurrRllldrrrddrrdllllUrrrruuuLrdddllllUdrruuluRllldRRllurrrDllddrrrruuuLrdddlllluurrDDrrdlLrrdlllluRRlldrrrruulllluuruRllldrrrddrrdLrdllllurRlldrrrruuuuuLrdddlllldrrdrUrdllllurrulluuruRllldrrrddlldrrrruLrdllllurRlldrrrruuuuLrdddLrdlllluuuruRllldrrrdDrruuuLrdddlllluuruRllldrrrddrrdlLrrdlllluurrrruuuLrdddllllddrrrrullLrrrdllllUrrrrulluulldDrrrruuulLrrdddlllluulurRRllldrrrddrrdllldlUrrrruuuuLrdddllldrrrdlllluUrrrruuulLrrdddlluulllurRRllldrrrddrrdlllluUdrrrruuuLrdddlllluUruRldrddrrdlllluuuluR
Runtime of dfs: 0.78 second.

$ python sokoban.py -l test8.txt -m dfs
llDlurrrdLrullldRDDDDDlllddrrdrruuLrddllulluurrruuuuulurrrdLrullldRdddddlllddrrUruLruuuuulurrrdLrullDDDDDDlddlluuRRllddrrdrruuLrddllulluurrDrUldrrddllUlluurrrUdlllddrrUruLruUddldrrddllulluuRlddrrdrruuluLruuUdddldrrddllulluuRlddrrdrruuluLruuuUddddldrrddllulluuRlddrrdrruuluLruuuuUluRldrdddddldrrddllulluuRRllddrrdrruulUUUUUU
Runtime of dfs: 0.11 second.

level1.txt: more than 1 minute.
```

* UCS：

```
$ python sokoban.py -l test1.txt -m ucs
rURdrUUlLdlU
Runtime of ucs: 0.09 second.

$ python sokoban.py -l test2.txt -m ucs
UUddrrrUU
Runtime of ucs: 0.01 second.

$ python sokoban.py -l test3.txt -m ucs
LrdrddDLdllUUdR
Runtime of ucs: 0.17 second.

$ python sokoban.py -l test4.txt -m ucs
llldRRR
Runtime of ucs: 0.01 second.

test5.txt: more than 1 minute.

$ python sokoban.py -l test6.txt -m ucs
dlluRdrUUUddrruuulL
Runtime of ucs: 0.02 second.

$ python sokoban.py -l test7.txt -m ucs
LUUUluRddddLdlUUUUluR
Runtime of ucs: 0.94 second.

$ python sokoban.py -l test8.txt -m ucs
llDDDDDDldddrruuLuuuuuuurrdLulDDDDDDlllddrrUdlluurRdddrruuLUUUUUUluRdddddddrddlluUdlluurRdrUUUUUU
Runtime of ucs: 0.31 second.

$ python sokoban.py -l level1.txt -m ucs
RurrddddlDRuuuuLLLrdRDrddlLdllUUdR
Runtime of ucs: 33.22 second.
```

* A star

```
$ python sokoban.py -l test1.txt -m astar
rUUdRdrUUluL
Runtime of astar: 0.01 second.

$ python sokoban.py -l test2.txt -m astar
UUddrrrUU
Runtime of astar: 0.01 second.

$ python sokoban.py -l test3.txt -m astar
LrdrddDLdllUUdR
Runtime of astar: 0.01 second.

$ python sokoban.py -l test4.txt -m astar
llldRRR
Runtime of astar: 0.00 second.

$ python sokoban.py -l test5.txt -m astar
uruLdlUURUdRdrUUllLdlU
Runtime of astar: 0.11 second.

$ python sokoban.py -l test6.txt -m astar
dlluRdrUUUddrruuulL
Runtime of astar: 0.02 second.

$ python sokoban.py -l test7.txt -m astar
LUUUluRddddLdlUUUUluR
Runtime of astar: 0.16 second.

$ python sokoban.py -l test8.txt -m astar
llDDDDDDldddrruuLuuuuuuurrdLulDDDDDDlllddrrUdlluurRdddrruuLUUUUUUluRdddddddrddlluUdlluurRdrUUUUUU
Runtime of astar: 0.33 second.

$ python sokoban.py -l level1.txt -m astar
RurrdLLLrrrdddlDRlLdllUUdRRurruuulldRDrddL
Runtime of astar: 0.85 second.
```

表現最好的是A*。BFS、UCS和A*輸出的推箱者動作一致，DFS雖然也能找到解決方式但會出現逛該現象，顯然其他方法會讓結果更簡單一些。後續可以提升的方面有：

* 定義更簡單的SSG
* 更加全面的dead pattern
* 更加適合的cost function和heuristic function
