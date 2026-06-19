## 春季class4

题目主要是打暴力。(后面因为纯纯的dfs暴力实在不好找又加了一些分块暴力)

有的题目是本来就是暴力题，有的则是可以暴力拿小评测用例的部分分（这些题我们今天只管部分分，所以不用在意题目的颜色）。

本讲义代码中，可能含有笔者的一些习惯性的`define`，大致如下：

```cpp
using ll = long long;
using ull = unsigned long long;
using lll = __int128;
using pll = pair<ll, ll>;
#define fp(i, l, r) for(int i=(l); i<=(r); ++i)
#define fq(i, r, l) for(int i=(r); i>=(l); --i)
#define fi first
#define se second
```

### 第一题： [蓝桥杯 2025 省 A  扫地机器人](https://www.luogu.com.cn/problem/P12145)

去年蓝桥杯的压轴题，我们只拿部分分就跑路。

看一眼小评测用例的范围。

**对于 20% 的评测用例，1≤*n*≤5000；**

那还说什么了，只要搞个复杂度$O(n^{2})$的做法，部分分不就到手了吗？

第一想法，肯定是考虑对每个点做起点，都跑一遍$dfs$，然后答案取最大值。

对这个做法进行一个复杂度的分析。因为图中有$n$个点$n$条边且保证联通，所以图中只有一个环。

那么在每个$dfs$的过程中，每条边最多经过两次，最多经过$2n$条边，复杂度$O(n)$，可行。

要注意一个点，主播第一次写的时候惯性思维给点标$vis$了然后$WA$，但实际上这题点是可以重复走的，是边只能走一次，所以要给边开一个$vis$。

然而，主播改完依旧$WA$了，因为改成了只对边开$vis$，重复走的点会重复贡献。

```cpp
int n;
int ans[N]; 
int t[N];
vector<pair<int,int>> G[N];
bool vis1[N];  // 边的vis
int vis2[N];  // 点的vis

int dfs(int u){
    int maxn=0;
    vis2[u]++;
    fp(i,0,G[u].size()-1){
        int v=G[u][i].fi;
        int id=G[u][i].se;
        if(vis1[id]) continue;
        vis1[id]=1;
        maxn=max(maxn,dfs(v));
        vis1[id]=0;
    }
    vis2[u]--;
    return (vis2[u] ? 0:t[u])+maxn;
}

void solve(){
    cin>>n;
    fp(i,1,n) cin>>t[i];
    fp(i,1,n){
        int u,v;
        cin>>u>>v;
        G[u].push_back({v,i});
        G[v].push_back({u,i});
    }
    fp(i,1,n) ans[i]=dfs(i);
    int res=0;
    fp(i,1,n) res=max(res,ans[i]);
    cout<<res;
}
```

![image-20260314175438039](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20260314175438039.png)

可以看到，跑出来的时间是很紧的，给了1s，跑了990ms。所以打完暴力没事干可以想办法优化一下常数。



### 第二题： [蓝桥杯 2025 省 B 第二场 翻转硬币](https://www.luogu.com.cn/problem/P12349)

依旧拿部分分跑路（

**对于 30% 的评测用例，*n*,*m*≤20；**

如果直接枚举每一行的翻转情况，然后对每个点进行计算的话，复杂度会到达$O(2^{n}*nm)$，很难不T。

这时候我们可以先预处理最开始的答案，然后每次翻转更新答案，这样的复杂度是$O(2^{n}*m)$，完工。

```cpp
ll ans=0;
ll maxn=0;
int n,m;
string s[N];
int a[N][N];
int cnt[N][N];

void dfs(int dep){
    if(dep==n+1) {
        maxn=max(maxn,ans);
        return;
    }

    dfs(dep+1); // 直接不翻这一行

    // 翻这一行的更新
    ll preans=ans;
    fp(i,1,m){
        int delta=0; // 对当前格子的影响
        if(dep!=1){
            if(a[dep][i]!=a[dep-1][i]) {
                delta++;
                ans+=(cnt[dep-1][i]+1)*(cnt[dep-1][i]+1)-cnt[dep-1][i]*cnt[dep-1][i];
                cnt[dep-1][i]++;
            }
            else{
                delta--;
                ans+=(cnt[dep-1][i]-1)*(cnt[dep-1][i]-1)-cnt[dep-1][i]*cnt[dep-1][i];
                cnt[dep-1][i]--;
            }
        }
        if(dep!=n){
            if(a[dep][i]!=a[dep+1][i]) {
                delta++;
                ans+=(cnt[dep+1][i]+1)*(cnt[dep+1][i]+1)-cnt[dep+1][i]*cnt[dep+1][i];
                cnt[dep+1][i]++;
            }
            else{
                delta--;
                ans+=(cnt[dep+1][i]-1)*(cnt[dep+1][i]-1)-cnt[dep+1][i]*cnt[dep+1][i];
                cnt[dep+1][i]--;
            }
        }
        ans+=(cnt[dep][i]+delta)*(cnt[dep][i]+delta)-cnt[dep][i]*cnt[dep][i];
        cnt[dep][i]+=delta;
    }
    fp(i,1,m) a[dep][i]=1-a[dep][i];
    dfs(dep+1);
    //回退
    ans=preans;
    fp(i,1,m){
        int delta=0;
        if(dep!=1){
            if(a[dep][i]!=a[dep-1][i]) {
                delta++;
                cnt[dep-1][i]++;
            }
            else{
                delta--;
                cnt[dep-1][i]--;
            }
        }
        if(dep!=n){
            if(a[dep][i]!=a[dep+1][i]) {
                delta++;
                cnt[dep+1][i]++;
            }
            else{
                delta--;
                cnt[dep+1][i]--;
            }
        }
        cnt[dep][i]+=delta;
    }
    fp(i,1,m) a[dep][i]=1-a[dep][i];
}

void solve(){
    cin>>n>>m;
    fp(i,1,n) cin>>s[i];
    fp(i,1,n) fp(j,0,m-1) {
        if(s[i][j]=='0') a[i][j+1]=0;
        else a[i][j+1]=1;
    }
    fp(i,1,n) fp(j,1,m){
        if(i!=1 and a[i-1][j]==a[i][j]) cnt[i][j]++;
        if(i!=n and a[i+1][j]==a[i][j]) cnt[i][j]++;
        if(j!=1 and a[i][j-1]==a[i][j]) cnt[i][j]++;
        if(j!=m and a[i][j+1]==a[i][j]) cnt[i][j]++;
        ans+=cnt[i][j]*cnt[i][j];
    }
    dfs(1);
    cout<<maxn;
}
```



### 第三题： [蓝桥杯 2023 省 Java A 与或异或](https://www.luogu.com.cn/problem/P13877)

这题本来就是填空题，本质上是有10个空，每个空可以填三种运算，dfs跑之，复杂度$O(3^n*10)$。

写一些工具类的函数可以减少点工作量。

```cpp
int op[11];
int res[11];
int cnt=0;

int ope(int a,int b,int x){
    if(x==1) return (a&b);
    if(x==2) return (a|b);
    if(x==3) return (a^b);
}

void dfs(int dep){
    if(dep==11){
        res[1]=ope(1,0,op[1]);
        res[2]=ope(0,1,op[2]);
        res[3]=ope(1,0,op[3]);
        res[4]=ope(0,1,op[4]);
        res[5]=ope(res[1],res[2],op[5]);
        res[6]=ope(res[2],res[3],op[6]);
        res[7]=ope(res[3],res[4],op[7]);
        res[8]=ope(res[5],res[6],op[8]);
        res[9]=ope(res[6],res[7],op[9]);
        res[10]=ope(res[8],res[9],op[10]);
        if(res[10]==1) cnt++;
        return;
    }
    op[dep]=1;
    dfs(dep+1);
    op[dep]=2;
    dfs(dep+1);
    op[dep]=3;
    dfs(dep+1);
}

void solve(){
    dfs(1);
    cout<<cnt;
}
```



### 第四题： [蓝桥杯 2024 国 Python B 不同的总分值](https://www.luogu.com.cn/problem/P12266)

填空题，每个数有选和不选两种状态，总共$2^{10}$种，我们用$set$去重。

```cpp
int a[11]={0,5,5,10,10,15,15,20,20,25,25};
set<int> ans;
int cur=0;

void dfs(int dep){
    if(dep==11){
        ans.insert(cur);
        return;
    }
    dfs(dep+1);
    cur+=a[dep];
    dfs(dep+1);
    cur-=a[dep];
}

void solve(){
    dfs(1);
    cout<<ans.size();
}
```



### 第五题：[蓝桥杯 2023 省 Python A 分糖果 - 洛谷](https://www.luogu.com.cn/problem/P13886?contestId=314842)

不考虑糖果够不够的话，每个孩子有$18$种选法，$18^{7}=6e8$，另外可以用糖果数量小于0的情况直接结束剪枝。

```cpp
int a=9;
int b=16;
ll cnt=0;

void dfs(int dep){
    if(a<0 or b<0) return;
    if(dep==8){
        if(a==0 and b==0) cnt++;
        return;
    }
    fp(i,0,5) fp(j,0,5){
        if(i+j<2 or i+j>5) continue;
        a-=i;
        b-=j;
        dfs(dep+1);
        a+=i;
        b+=j;
    }
}

void solve(){
    // dfs(1);
    // cout<<cnt;
    cout<<5067671;
}
```



### 第六题： [**Team Building**](https://atcoder.jp/contests/awc0017/tasks/awc0017_d)

每个人有选和不选两种状态，共$2^{n}$种选法，每种选法要遍历$m$对不兼容对，复杂度$O(2^{n}*m)$。

```cpp
const int N = 19;
const int M = 200;

ll n,m,k;
vector<ll> a(N);
vector<bool> vis(N);
vector<array<ll,3>> p(M);
ll ans=-INF;
ll cur=0;

void dfs(ll dep,ll cnt){
    if(dep==n+1){
        if(cnt!=k) return;
        ll tmp=0;
        fp(i,1,m) if(vis[p[i][0]] && vis[p[i][1]]) tmp+=p[i][2];
        ans=max(ans,cur-tmp);
        return;
    }
    dfs(dep+1,cnt);
    if(cnt==k) return;
    cur+=a[dep];
    vis[dep]=1;
    dfs(dep+1,cnt+1);
    vis[dep]=0;
    cur-=a[dep];
}

void solve(){
    cin>>n>>m>>k;
    fp(i,1,n) cin>>a[i];
    fp(i,1,m) cin>>p[i][0]>>p[i][1]>>p[i][2];
    dfs(1,0);
    cout<<ans;
}
```



### 第七题：[**E - Exhibition Booth Arrangement**](https://atcoder.jp/contests/awc0010/tasks/awc0010_e)

总共有 $n!$ 种排序方法，对于每一种，我们可以$O(n^{2})$检查这种排序方法能否在$k$次以内换到，如果能，$O(n)$计算总值，对于所有的总值取最大。复杂度$O(n!*n^{2})$。

```cpp
void solve(){
    ll n,k;
    cin>>n>>k;
    vector c(n+1,vector<ll>(n+1));
    for(int i=1;i<=n;++i){
        for(int j=1;j<=n;++j){
            cin>>c[i][j];
        }
    }
    vector<bool> vis(n+1);
    vector<ll> a(n+1);
    ll ans=0;

    auto check=[&]() ->bool {
        vector<ll> b(n+1);
        ll cnt=0;
        for(int i=1;i<=n;++i) b[i]=a[i];
        for(int i=1;i<=n;++i)
            if(b[i]!=i){
                cnt++;
                for(int j=i+1;j<=n;++j)
                    if(b[j]==i) swap(b[i],b[j]);
            }
        if(cnt<=k) return 1;
        return 0;
    };

    auto cal=[&]() ->ll {
        ll res=0;
        for(int i=1;i<=n-1;++i) res+=c[a[i]][a[i+1]];
        res+=c[a[n]][a[1]];
        return res;
    };

    auto dfs=[&](auto&& self, ll dep) {
        if(dep==n+1){
            if(check()){
                ans=max(ans,cal());
            }
            return;
        }
        for(int i=1;i<=n;++i){
            if(vis[i]) continue;
            vis[i]=1;
            a[dep]=i;
            self(self,dep+1);
            vis[i]=0;
        }
    };

    dfs(dfs,1);
    cout<<ans;
}
```



### 第八题：[**E - Assortment of Sweets**](https://www.luogu.com.cn/problem/AT_awc0002_e?contestId=314842)

这题很有意思。

没看数据范围之前一眼背包板题，结果一看数据范围，值域$10^{12}$，直接吓哭了。

好消息是$n$只有$40$，看起来能搜索。坏消息是$2^{40}=10^{12}$，感觉怎么剪枝都会T。

你回想前面做的这么多的爆搜题，总觉得正确的复杂度应该是$2^{20}$才对，即$2^{\frac{n}{2}}$。

$\frac{n}{2}$是什么东西呢，是跑一半的物品吗？那么另一半怎么办，也跑一遍吗？想到这里大概就有思路了。

把$n$个物品拆成两半，分别跑$dfs$，然后把两边的信息组合起来。

```cpp
vector<ll> a(41);
ll n,s;
ll now=0;
ll cnt=0;
ll u;
ll v;
map<ll,ll> cnt1;
map<ll,ll> cnt2;

void dfs1(ll dep){
    if(dep==u+1){
        cnt1[now]++;
        return;
    }
    now+=a[dep];
    dfs1(dep+1);
    now-=a[dep];
    dfs1(dep+1);
}

void dfs2(ll dep){
    if(dep==n+1){
        cnt2[now]++;
        return;
    }
    now+=a[dep];
    dfs2(dep+1);
    now-=a[dep];
    dfs2(dep+1);
}

void solve(){
    cin>>n>>s;
    u=n/2;
    v=n-u;
    for(int i=1;i<=n;++i){
        cin>>a[i];
    }
    now=0;
    dfs1(1);
    now=0;
    dfs2(u+1);
    ll ans=0;
    for(auto &[x,y]:cnt1){
        ans+=cnt2[s-x]*y;
    }
    cout<<ans;
}
```

### 第九题：[E - Cargo Delivery Truck](https://atcoder.jp/contests/awc0003/tasks/awc0003_e)

依旧考虑搜索。

理论上搜索能达到$15^{15}=4e17$，感觉也是怎么都会T。

但是，这题确实能是剪枝过去的，剪枝很神秘吧（

因为只要有一种方案成功了，我们就可以判断可行，然后不用跑了，所以我们尽量让更可能成功的方案早点出现。

为了做到这一点，我们要先装承载量大的卡车，即按承载量大降序排序卡车。

或者如果前几个包裹就已经无论如何都装不了了，那么我们就可以判断绝对没有可行方案，后面也不用跑了。

为了做到这一点，我们要先装重的包裹，即按重量降序排序包裹。

另外，我们还可以提前判断两种不可能有可行方案的情况，即最重的包裹重于承载量最大的卡车，和包裹的总重量大于卡车的总承载量。

在这些神秘剪枝的帮助下，我们可以顺利通过此题。

```cpp
vector<ll> w(16);
vector<ll> c(16);
ll n,m;

bool dfs(ll dep){
    if(dep==n+1){
        return 1;
    }
    for(int i=1;i<=m;++i){
        if(c[i]>=w[dep]){
            c[i]-=w[dep];
            bool flag=dfs(dep+1);
            if(flag) {return 1;}
            c[i]+=w[dep];
        }
    }
    return 0;
}

void solve(){
    cin>>n>>m;
    ll sw=0;
    ll sc=0;
    ll maxw=0;
    ll maxc=0;
    for(int i=1;i<=n;++i) {cin>>w[i];sw+=w[i];maxw=max(maxw,w[i]);}
    sort(w.begin()+1,w.begin()+n,greater<ll>());
    for(int j=1;j<=m;++j) {cin>>c[j];sc+=c[j];maxc=max(maxc,c[j]);}
    sort(c.begin()+1,c.begin()+m,greater<ll>());
    if(sw>sc or maxw>maxc){
        cout<<"No";
        return;
    }

    bool flag=dfs(1);
    if(flag) cout<<"Yes";
    else cout<<"No";
}
```

### 第十题：[**E - Count the Types of Flowers**](https://atcoder.jp/contests/awc0015/tasks/awc0015_e)

好玩而又暴力的题目。

如果我们已经知道了区间**[l,r]**中 不同类型的花卉的个数以及每种花卉的数量 的话，我们是可以$O(1)$得到**[l+1,r]**，**[l-1,r]**，**[l,r-1]**和**[l,r+1]**这四个区间之一的 不同类型的花卉的个数以及每种花卉的数量 的。

所以，对于每个查询，我们都可以通过移动前一个查询的左右端点（即 $l$ 和 $r$ ），来得到下一个查询的答案。

然而，这样子的最坏情况，每次我们都要移动$O(n)$次端点，才能到达下一个查询的区间。

为了减少端点的移动次数，我们可以考虑把查询换一下顺序，即先把所有的查询存下来，对这些查询按照我们希望的顺序排序以减少端点的移动次数。完成所有的查询后再按照题目要求的顺序输出。

那么怎么样才可以减少移动次数呢，我们考虑分块，即将n分为$\sqrt n$个块，每个块长度为$\sqrt n$。

然后我们按照这样的方法对查询[l,r]排序：

1. 先比较左端点 $l$ 所在的块，让左端点 $l$ 所在的块更靠左的排在前面。
2. 如果左端点所在的块一样，那么如果左端点在偶数块，就按照 $r$ 递增地排序，如果左端点在奇数块，就按照 $r$ 递减地排序。

我们现在证明，按照这个排序方法，端点移动次数不超过$O(n\sqrt n+q\sqrt n)$。

对于左端点$l$，它有两种移动，即块内移动和跨块移动。

对于块内移动，每次最多移动$\sqrt n$，最多移动$q$次，即最多$q\sqrt n$。

对于跨块移动，每次最多移动$2\sqrt n$，最多跨$\sqrt n$次块，即最多$O(n)$。

对于右端点$r$，它对于每一个左端点所在的分块，都最多移动$n$次（即从最左移到最右或最右移到最左），而左端点最多只在$\sqrt n$个块中，所以$r$最多移动$n\sqrt n$ 次。

```cpp
struct qry{
    ll l,r,block,i;
};

bool cmp(qry a,qry b){
    if(a.block!=b.block){
        return a.block<b.block;
    }
    else if(a.block%2==0) return a.r<b.r;
    else return a.r>b.r;
}

void solve(){
    ll n,q;
    cin>>n>>q;
    vector<ll> p(n);
    fp(i,0,n-1) cin>>p[i];
    vector<qry> qrys;
    ll d=(ll)sqrt(n);
    fp(i,1,q){
        ll l,r;
        cin>>l>>r;
        qrys.push_back({l-1,r-1,(l-1)/d,i});
    }
    sort(qrys.begin(),qrys.end(),cmp);

    vector<ll> ans(q+1);
    vector<ll> cnt(n+1);

    ll curl=0;
    ll curr=0;
    ll curn=1;
    cnt[p[0]]++;

    auto add=[&](ll x){
        if(cnt[p[x]]==0) curn++;
        cnt[p[x]]++;
    };

    auto remove=[&](ll x){
        if(cnt[p[x]]==1) curn--;
        cnt[p[x]]--;
    };

    fp(i,0,q-1){
        auto tar=qrys[i];
        ll l=tar.l;
        ll r=tar.r;
        ll id=tar.i;
        while(curl>l){
            add(curl-1);
            curl--;
        }
        while(curr<r){
            add(curr+1);
            curr++;
        }
        while(curl<l){
            remove(curl);
            curl++;
        }
        while(curr>r){
            remove(curr);
            curr--;
        }
        ans[id]=curn;
    }

    fp(i,1,q){
        cout<<ans[i]<<"\n";
    }
}
```



### 第十一题：[F - Authentic Traveling Salesman Problem](https://atcoder.jp/contests/abc448/tasks/abc448_f)

和第十题一样的分块思路。

```cpp
struct pos{
    ll x,y,bx,id;
};

bool cmp(pos a,pos b){
    if(a.bx!=b.bx){
        return a.bx<b.bx;
    }
    if(a.bx%2==0) return a.y<b.y;
    else return a.y>b.y;
}

void solve(){
    ll n;
    cin>>n;
    vector<pos> p(n-1);
    ll x,y;
    cin>>x>>y;
    if(n==1){
        cout<<1;
        return;
    }
    ll d=(ll)sqrt((ll)(4e14)/n);
    fp(i,0,n-2){
        cin>>x>>y;
        p[i]={x-1,y-1,(x-1)/d,i+2};
    }
    cout<<1<<" ";
    sort(p.begin(),p.end(),cmp);
    fp(i,0,n-2){
        cout<<p[i].id<<" ";
    }
}
```

