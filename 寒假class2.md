# 前言

笔者主要想分享一些基础的字符串，由于笔者也是现学的，所以难免会犯各种错误，如有不当之处，敬请指正。

同时，笔者在写讲义的过程中发现自己**完全**不擅长教学，所以本讲义含有大量的复制粘贴，面向AI，以及直接贴链接，敬请见谅。

# 字符串哈希

## 引入

我们现在有2个同样长的字符串，我们希望比较它们是否相同，该怎么办呢？

最直观的方法，就是一个字符一个字符地比较，时间复杂度$O(len)$，$len$为字符串的长度。

那如果现在有n个字符串，希望找到所有相同的字符串呢？

如果依旧按照朴素方法，每次比较都要一个一个字符地比，一旦字符串比较长，复杂度就会很大。

那么，有没有$O(1)$比较字符串的方法呢？

有的，兄弟有的。

## 哈希函数

我们知道，比较两个整数是否相同的时间复杂度为$O(1)$，那么如果可以把不同的字符串对应不同的整数，我们就只需要比较对应的数就行了。
而哈希函数，就是一种把字符串映射到整数的函数。

我们不妨先考虑只有单个字符的情况，可以想到，直接使用字符对应的$ASCII$码就行了。

那么现在面对字符串了，我们希望有一种办法，可以将几个数组合成一个数，且互不影响。（或许）可以想到，直接把数字拼在一起就行了。

于是我们得到了最常用的多项式 Hash 函数：

$$
f(s) = \sum_{i=1}^{l} s[i] \cdot b^{l-i} \pmod M
$$

（b 大于所有 s[i] 的 ASCII 码最大值，M 为大质数）

我们可以把它理解为一个长为$l$的$b$进制数。这个$b$进制数的第$i$位，就是字符串中的第$i$位的$ASCII$码。

如果先不考虑取模，那么

两个字符串相同$\Leftrightarrow$

两个字符串中的字符都相同$\Leftrightarrow$

两个$b$进制数的各位都相同$\Leftrightarrow$

两个$b$进制数相同

同理，两个字符串不相同$\Leftrightarrow$两个$b$进制数不相同

## 哈希碰撞（冲突）

但是，由于得到的数可能很大，所以我们不得不为这个数取模，这就使得两个不同的字符串的$Hash$值可能相同。

我们将$Hash$值相同但原字符串不相同的现象称为**哈希碰撞**。

值得区分的是，虽然取模导致了$Hash$值相同的时候，两个字符串不一定相同（但是大概率相同）。

但是$Hash$值不相同时，取模之前的数一定也不相同，继而两个字符串肯定不相同。

为避免哈希碰撞，我们希望模数$M$有以下特征：

1.大，这样区间更大，能生成的不同哈希值更多；

2.$M\in\mathbb{P}$，这样$M$的因子最少，取模后结果相等的可能性最小。

所以，我们一般选取$998244353$，$1e9+7$这样的数作为模数。

还有一种写法，为了方便和扩大模数，我们可以使用unsigned long long来定义$Hash$函数的结果，不进行取模。这种写法被称为**自然溢出**。

ull在$2^{64}-1$再$+1$后会溢出，直接从$0$重新开始，所以我们相当于把模数$M$定为$2^{64}$，这也是一个不错的选择。

## 参考资料

$Hash$的技巧还有很多，感兴趣的可以看看以下参考资料。

[OI Wiki 字符串哈希](https://oi-wiki.org/string/hash/)


## 第一题：P3370【模板】字符串哈希

模板题喵。

```cpp
const ll MOD = 998244353;
const ll base = 191;
ll has(string &s){
    ll len=s.size();
    ll ans=0;
    for(char c:s){
        ans=(ans*base+c-'a'+1)%MOD;
    }
    return ans;
}

void solve(){
    ll n;
    cin>>n;
    vector<ll> vec;
    for(int i=1;i<=n;++i){
        string s;
        cin>>s;
        vec.push_back(has(s));
    }
    sort(vec.begin(),vec.end());
    vec.erase(unique(vec.begin(),vec.end()),vec.end());
    cout<<vec.size();
}
```

## 第二题：P3879 阅读理解

set太好用了你们知道吗？

```cpp
const ll MOD = 998244353;
const ll base = 191;
ll has(string s){
    ll len=s.size();
    ll ans=0;
    for(char c:s){
        ans=(ans*base+c-'a'+1)%MOD;
    }
    return ans;
}

void solve(){
    ll n;
    cin>>n;
    vector<set<ll>> a(n+1);
    for(int i=1;i<=n;++i){
        ll l;
        cin>>l;
        for(int j=1;j<=l;++j){
            string s;
            cin>>s;
            a[i].insert(has(s));
        }
    }
    ll m;
    cin>>m;
    while(m--){
        string s;
        cin>>s;
        ll t=has(s);
        for(int i=1;i<=n;++i){
            if(a[i].find(t)!=a[i].end()){
                cout<<i<<" ";
            }
        }
        cout<<"\n";
    }
}
```

# 字典树

## 参考资料

[Trie树的可视化操作](https://gallery.selfboot.cn/zh/algorithms/trie)

[字典树 (Trie) - OI Wiki](https://oi-wiki.org/string/trie/)

[算法学习笔记(44): 01字典树](https://zhuanlan.zhihu.com/p/174830050)

## 第三题：[P8306【模板】字典树](https://www.luogu.com.cn/problem/P8306)

我这个纯为了做题写的，板子的话还是建议学习丁冠，搞结构体封装。

```cpp
vector<map<ll,ll>> nxt(N);
vector<ll> vis(N);

void solve(){
    ll n,q;
    cin>>n>>q;
    ll cnt=1;

    for(int i=1;i<=n;++i){ //建树
        string s;
        cin>>s;
        ll len=s.size();
        s=" "+s;
        ll tmp=0;
        for(int j=1;j<=len;++j){
            ll x=s[j]-'0';
            if(!nxt[tmp][x]){
                nxt[tmp][x]=cnt;
                cnt++;
            }
            tmp=nxt[tmp][x];
            vis[tmp]++;
        }
    }

    while(q--){ //查询
        string s;
        cin>>s;
        ll len=s.size();
        s=" "+s;
        ll tmp=0;
        for(int i=1;i<=len;++i){
            ll x=s[i]-'0';
            if(!nxt[tmp][x]){
                cout<<0<<"\n";
                break;
            }
            tmp=nxt[tmp][x];
            if(i==len){
                cout<<vis[tmp]<<"\n";
                break;
            }
        }
    }

    for(int i=0;i<=cnt-1;++i){
        nxt[i].clear();
        vis[i]=0;
    }
}
```
## 第四题：[P10471 最大异或对](https://www.luogu.com.cn/problem/P10471)

对于一个确定的数，要找和它异或结果最大的数，我们只需要从它的二进制最高位开始，如果能取相反的位，那么肯定是取了的结果比不取的更大，如果不能取才取相同的位。

而取尽可能取相反的位这个操作正好可以用字典树来实现。

考虑将$A_1$，$A_2$，…，$A_n$转化为31位的2进制数，按照从位数从大到小插入字典树。

然后对于每个$A_i$，我们都希望在每一位上尽可能取与$A_i$该位相反的数。对应在字典树上，就是判断由当前位置指向与$A_i$该位相反的数的路径是否存在，存在就选相反的数，不存在就取相同的数。通过不断地重复这样的操作，我们可以找出与$A_i$异或结果最大的数，继而得到选了$A_i$的最大的异或结果。

$n$个结果取最大值即答案。

```cpp
vector<vector<ll>> nxt(N,vector<ll>(2,0));
void solve(){
    ll n;
    cin>>n;
    vector<vector<int>> a(n+1,vector<int>(32));
    for(int i=1;i<=n;++i) {
        ll x;
        cin>>x;
        for(int j=31;j>=1;--j){
            a[i][j]=x&1;
            x>>=1;
            
        }
    }
    ll cnt=1;
    for(int i=1;i<=n;++i){ //建树
        ll tmp=0;
        for(int j=1;j<=31;++j){
            ll x=a[i][j];
            if(!nxt[tmp][x]){
                nxt[tmp][x]=cnt;
                cnt++;
            }
            tmp=nxt[tmp][x];
        }
    }

    ll ans=0;
    for(int i=1;i<=n;++i){
        ll res=0;
        ll tmp=0;
        for(int j=1;j<=31;++j){
            ll x=a[i][j];
            if(nxt[tmp][1-x]){
                res+=(1LL<<(31-j));
                tmp=nxt[tmp][1-x];
            }
            else{
                tmp=nxt[tmp][x];
            }
        }
        ans=max(ans,res);
    }
    cout<<ans;
}
```
## 第五题：[P4551 最长异或路径](https://www.luogu.com.cn/problem/P4551)

由于异或的神秘小性质$a \oplus a=0$，（或许）可以想到，$x$和$y$之间的路径的异或值其实等于 $1$和$x$之间的路径的异或值 异或上 $1$和$y$之间的路径的异或值。因为这样重复的路径会抵消掉。

然后就变成和第四题一样了，先跑一遍dfs算出1到每个点的路径异或值，然后就是找这些里面的最大异或对。

```cpp
vector<vector<pll>> G(N);
vector<ll> dis(N);
vector<bool> vis(N);

void dfs(ll u,ll x){
    vis[u]=1;
    dis[u]=x;
    for(auto [v,w]:G[u]){
        if(vis[v]) continue;
        dfs(v,(x^w));
    }
}

void solve(){
    ll n;
    cin>>n;
    for(int i=1;i<=n-1;++i){
        ll u,v,w;
        cin>>u>>v>>w;
        G[u].push_back({v,w});
        G[v].push_back({u,w});
    }
    dfs(1,0);
    vector<vector<int>> a(n+1,vector<int>(32));
    for(int i=1;i<=n;++i) {
        ll x=dis[i];
        for(int j=31;j>=1;--j){
            a[i][j]=(x&1);
            x>>=1;
            
        }
    }

    vector<vector<ll>> nxt(n*31,vector<ll>(2,0));
    ll cnt=1;
    for(int i=1;i<=n;++i){ //建树
        ll tmp=0;
        for(int j=1;j<=31;++j){
            ll x=a[i][j];
            if(!nxt[tmp][x]){
                nxt[tmp][x]=cnt;
                cnt++;
            }
            tmp=nxt[tmp][x];
        }
    }

    ll ans=0;
    for(int i=1;i<=n;++i){
        ll res=0;
        ll tmp=0;
        for(int j=1;j<=31;++j){
            ll x=a[i][j];
            if(nxt[tmp][1-x]){
                res+=(1LL<<(31-j));
                tmp=nxt[tmp][1-x];
            }
            else{
                tmp=nxt[tmp][x];
            }
        }
        ans=max(ans,res);
    }
    cout<<ans;
}
```

# KMP

## 参考资料

本来$KMP$想自己写的，但我写着写着发现$KMP$不配图的话我确实讲的不怎么清楚，建议大家都看参考资料喵。

[非常好的视频](https://www.bilibili.com/video/BV1Er421K7kF/?spm_id_from=333.337.search-card.all.click&vd_source=420f4ec974894ef0305547b9645d494d)

[以及一个很好的俄语博文的翻译](https://oi-wiki.org/string/kmp/)


## 第六题:【模板】KMP

```cpp
vector<ll> ans;

void kmp(string a,string b){
    string s=" "+b+"#"+a;
    vector<ll> pi(s.size(),0);
    for(int i=2;i<=s.size()-1;++i){
        ll len=pi[i-1];
        while(len and s[i]!=s[len+1]){
            len=pi[len];
        }
        if(s[i]==s[len+1]){
            pi[i]=len+1;
        }
        if(pi[i]==b.size()){
            ans.push_back(i-2*b.size());
        }
    }
}

void solve(){
    string a,b;
    cin>>a>>b;
    kmp(a,b);
    for(auto &p:ans) cout<<p<<"\n";
    b=" "+b;
    vector<ll> pi(b.size(),0);
    cout<<0<<" ";
    for(int i=2;i<=b.size()-1;++i){
        ll len=pi[i-1];
        while(len and b[i]!=b[len+1]){
            len=pi[len];
        }
        if(b[i]==b[len+1]){
            pi[i]=len+1;
        }
        cout<<pi[i]<<" ";
    }
}
```

## 第七题:[P9606 ABB](https://www.luogu.com.cn/problem/P9606?contestId=306697)

容易想到，只要求出了包含最后一个字符的最长回文串的长度$len$，$n-len$即是答案。

那么怎么求呢？

考虑将字符串$s$反转得到$t$，拼接字符串$t$+"\#"+$s$得到$k$，$k$的最后一个字符的前缀函数的值，就是包含最后一个字符的最长回文串的长度。

有点绕，举个栗子。比如$s$的长度为4，我们设$s$为abcd,那么生成的$k$为dcba\#abcd，为了方便，我们称$k$的最后一个字符为$x$。然后比如我们得到$x$处的前缀函数的值为3，说明dcb和bcd是相同的，即bcd是回文串,这说明了存在性。然后如果还有更长的回文，那么$x$处的前缀函数的值会大于3，矛盾，所以包含$x$的最长回文串的长度就是$x$处的前缀函数的值。

但我个人认为这个$KMP$的做法不太好想，而用下一节的$Manacher$则会好想很多，因为$Manacher$本身就是求回文串的算法。

```cpp
vector<ll> ans;
ll maxn;

void kmp(string a,string b){
    string s=" "+b+"#"+a;
    vector<ll> pi(s.size(),0);
    for(int i=2;i<=s.size()-1;++i){
        ll len=pi[i-1];
        while(len and s[i]!=s[len+1]){
            len=pi[len];
        }
        if(s[i]==s[len+1]){
            pi[i]=len+1;
        }
        if(pi[i]==b.size()){
            ans.push_back(i-2*b.size());
        }
    }
    maxn=pi[s.size()-1];
}

void solve(){
    ll n;
    cin>>n;
    string s,t;
    cin>>s;
    t=s;
    reverse(t.begin(),t.end());
    kmp(s,t);
    cout<<n-maxn;
}
```

## 第八题：[P4824 [USACO15FEB] Censoring S](https://www.luogu.com.cn/problem/P4824?contestId=306697)

稍微修改一下$KMP$的板子，因为会有删除，所以改用类似栈的形式写（不用栈是因为栈只能取栈顶的值）。每次`pi[tmp]==b.size()`就回退。

```cpp
void kmp(string a,string b){
    string s1=" "+b+"#"+a;
    vector<ll> pi(2,0);
    vector<char> s(1);
    s.push_back(s1[1]);
    ll tmp=1;
    for(int i=2;i<=s1.size()-1;++i){
        ll len=pi[tmp];
        tmp++;
        while(len and s1[i]!=s[len+1]){
            len=pi[len];
        }
        if(s1[i]==s[len+1]){
            pi.push_back(len+1);
        }
        else{
            pi.push_back(0);
        }
        s.push_back(s1[i]);
        if(pi[tmp]==b.size()){
            for(int i=1;i<=b.size();++i){
                pi.pop_back();
                s.pop_back();
            }
            tmp-=b.size();
        }
    }
    for(int i=b.size()+2;i<=s.size()-1;++i) cout<<s[i];
}

void solve(){
    string a,b;
    cin>>a>>b;
    kmp(a,b);
}
```

# Manacher

## 参考资料

[OI Wiki Manacher](https://oi-wiki.org/string/manacher/)

## 第九题:P3805【模板】Manacher

```cpp
void solve(){
    string s1;
    cin>>s1;
    int len=s1.size();
    vector<int> p(2*len+2);
    vector<char> s(2*len+2);
    s[0]=' ';
    s[1]='*';
    for(int i=0;i<=len-1;++i){
        s[2+2*i]=s1[i];
        s[3+2*i]='*';
    }
    int maxpos=0;
    int mid=0;
    for(int i=1;i<=2*len+1;++i){
        if(maxpos>=i){
            ll k=2*mid-i;
            p[i]=min(p[k],maxpos-i+1);
        }
        else{
            p[i]=1;
        }
        while(i-p[i]>=1 and i+p[i]<=len*2+1 and s[i-p[i]]==s[i+p[i]]){
            p[i]++;
        }
        if(i+p[i]-1>maxpos){
            maxpos=i+p[i]-1;
            mid=i;
        }
    }
    int ans=0;
    for(int i=1;i<=2*len+1;++i){
        ans=max(ans,(2*p[i]-1)/2);
    }
    cout<<ans;
}
```

## 第七题:P9606 ABB

回头一看，第六题用$Manacher$确实直观好想嘛。

现在只要先跑一遍$Manacher$，然后对于每个回文中心，判断以它为中心的回文串是否能达到最后一个字符，第一个能达到的就是最长回文串的长度。

```cpp
void solve(){
    int len;
    cin>>len;
    string s1;
    cin>>s1;
    vector<int> p(2*len+2);
    vector<char> s(2*len+2);
    s[0]=' ';
    s[1]='*';
    for(int i=0;i<=len-1;++i){
        s[2+2*i]=s1[i];
        s[3+2*i]='*';
    }
    int maxpos=0;
    int mid=0;
    for(int i=1;i<=2*len+1;++i){
        if(maxpos>=i){
            ll k=2*mid-i;
            p[i]=min(p[k],maxpos-i+1);
        }
        else{
            p[i]=1;
        }
        while(i-p[i]>=1 and i+p[i]<=len*2+1 and s[i-p[i]]==s[i+p[i]]){
            p[i]++;
        }
        if(i+p[i]-1>maxpos){
            maxpos=i+p[i]-1;
            mid=i;
        }
    }
    int ans=len;
    for(int i=len;i<=2*len+1;++i){
        if(i+p[i]-1>=2*len){
            ll x=(2*p[i]-1)/2;
            ans=len-x;
            break;
        }
    }
    cout<<ans;
}
```

## 第十题：[P1659 [国家集训队] 拉拉队排练](https://www.luogu.com.cn/problem/P1659?contestId=306697)

先跑一遍$Manacher$。

现在我们已经有了所有的回文串了，下一个目标就是找出最长的k个。

由于只要奇数长度的回文串，所以我们只用考虑回文串中心在原字符串上的回文串就行。

对于回文串中心$x$,如果最长回文串的半径长度为$r$,那么该中心总共可以贡献长度为2r-1,2r-3,…，1的回文串。观察这种形式，可以想到用差分来解决。对于每个中心，`diff[2*r-1]++`。然后对于差分数组，从最长开始做每次-2的后缀和，每次得到的就是当前长度的个数。不断相乘，直到总个数达到k。

```cpp
const ll MOD = 19930726;

ll qpow(ll a,ll b){
    ll res=1;
    while(b){
        if(b%2) res=res*a%MOD;
        b>>=1;
        a=a*a%MOD;
    }
    return res;
}

void solve(){
    ll n,k;
    cin>>n>>k;
    string s1;
    cin>>s1;
    int len=s1.size();
    vector<int> p(2*len+2);
    vector<char> s(2*len+2);
    s[0]=' ';
    s[1]='*';
    for(int i=0;i<=len-1;++i){
        s[2+2*i]=s1[i];
        s[3+2*i]='*';
    }
    int maxpos=0;
    int mid=0;
    for(int i=1;i<=2*len+1;++i){
        if(maxpos>=i){
            ll k=2*mid-i;
            p[i]=min(p[k],maxpos-i+1);
        }
        else{
            p[i]=1;
        }
        while(i-p[i]>=1 and i+p[i]<=len*2+1 and s[i-p[i]]==s[i+p[i]]){
            p[i]++;
        }
        if(i+p[i]-1>maxpos){
            maxpos=i+p[i]-1;
            mid=i;
        }
    }
    ll cnt=0;
    ll maxn=0;
    vector<ll> diff(N);
    for(int i=2;i<=2*len;i=i+2){
        ll x=(2*p[i]-1)/2;
        cnt+=x;
        diff[x]++;
        maxn=max(maxn,(ll)(2*p[i])-1);
    }
    if(cnt<k) cout<<-1<<"\n";
    else{
        ll ans=1;
        ll tmp=0;
        for(int i=maxn;i>=1;i=i-2){
            tmp+=diff[i];
            if(k>tmp){
                ans=ans*qpow(i,tmp)%MOD;
                k-=tmp;
            }
            else{
                ans=ans*qpow(i,k)%MOD;
                break;
            }
        }
        cout<<ans;
    }
}
```
## 第十一题：[P4555 [国家集训队] 最长双回文串](https://www.luogu.com.cn/problem/P4555?contestId=306697)

依旧先跑一遍$Manacher$。

考虑对于每个点，找它左边的最长回文串和右边的最长回文串。最长回文串又可以转化为回文串中心离该点最远。那么只需要对于每个回文串中心，在它最远达到的左右点处更新最远中心（当然如果不如原来远就不变了）。记得这样做之后要对右边最远跑一次前缀max,即`right[i]=max(right[i-1],right[i])`，因为我们只对最远的点更新了，而实际上右边的一个中心都能到你的左边了，当然更能到你这个点。左边同理也要跑一次后缀min。

```cpp
void solve(){
    string s1;
    cin>>s1;
    int len=s1.size();
    vector<int> p(2*len+2);
    vector<char> s(2*len+2);
    s[0]=' ';
    s[1]='*';
    for(int i=0;i<=len-1;++i){
        s[2+2*i]=s1[i];
        s[3+2*i]='*';
    }
    int maxpos=0;
    int mid=0;
    for(int i=1;i<=2*len+1;++i){
        if(maxpos>=i){
            ll k=2*mid-i;
            p[i]=min(p[k],maxpos-i+1);
        }
        else{
            p[i]=1;
        }
        while(i-p[i]>=1 and i+p[i]<=len*2+1 and s[i-p[i]]==s[i+p[i]]){
            p[i]++;
        }
        if(i+p[i]-1>maxpos){
            maxpos=i+p[i]-1;
            mid=i;
        }
    }
    vector<ll> left(2*len+2,INF);
    vector<ll> right(2*len+2,-INF);
    for(int i=1;i<=2*len+1;++i){
        left[i+(p[i]-1)]=min(left[i+(p[i]-1)],(ll)i);
        right[i-(p[i]-1)]=max(right[i-(p[i]-1)],(ll)i);
    }
    for(int i=1;i<=2*len+1;++i){
        right[i]=max(right[i-1],right[i]);
    }
    for(int i=2*len;i>=1;--i){
        left[i]=min(left[i+1],left[i]);
    }
    ll ans=0;
    for(int i=3;i<=2*len-1;i=i+2){
        ans=max(ans,right[i]-left[i]);
    }
    cout<<ans<<"\n";
}
```