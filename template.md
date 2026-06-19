[TOC]
# 0. 基础板子

```cpp
#include <bits/stdc++.h>
#include <bits/extc++.h>
using namespace std;
// using namespace __gnu_pbds;
using ll = long long;
using ull = unsigned long long;
using lll = __int128;
using pll = pair<ll, ll>;
#define fp(i, l, r) for(int i=(l); i<=(r); ++i)
#define fq(i, r, l) for(int i=(r); i>=(l); --i)
#define fi first
#define se second
#define debug(x) cerr << #x << ": " << x << '\n';
const int N = 2e5+10;
const ll MOD = 998244353;
const ll INF = 0x3f3f3f3f3f3f3f3f;
mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());

void solve(){
    
}

int main(){
    ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
    int _ =1;
    // cin>> _;
    while(_--)
        solve();
    return 0;
}
```



# 1. 数据结构

## 1.1 并查集

```cpp
struct DSU//启发式合并 + 路径压缩
{
    int n;
    vector<int> fa, sz;
    explicit DSU(int n) : n(n)
    {
        fa.assign(n + 1, 0);
        sz.assign(n + 1, 1);
        for (int i = 1; i <= n; i++)
            fa[i] = i;
    }
    void reset()
    {
        for(int i = 0; i <= n; ++i)
            fa[i] = i, sz[i] = 1;
    }
    int find(int x)
    {
        if(fa[x] == x)
            return x;
        return fa[x] = find(fa[x]);
    }
    void merge(int x, int y)
    {
        int dx = find(x);
        int dy = find(y);
        if(dx != dy)
        {
            if(sz[dx] > sz[dy])
                swap(dx, dy);
            sz[dy] += sz[dx];
            fa[dx] = dy;
        }
    }
    bool judge(int x, int y)
    {
        return find(x) == find(y);
    }
    int size(int x)
    {
        return sz[find(x)];    
    }
};
```

## 1.1.1 带权并查集

```cpp
struct WDSU
{
    int n;
    vector<int> fa, sz;
    vector<ll> d;
    explicit WDSU(int n) : n(n)
    {
        fa.assign(n + 1, 0);
        sz.assign(n + 1, 1);
        d.assign(n + 1, 0);
        for (int i = 1; i <= n; i++)
            fa[i] = i;
    }
    void reset()
    {
        for(int i = 0; i <= n; ++i)
            fa[i] = i, sz[i] = 1, d[i] = 0;
    }
    int find(int x)
    {
        if(fa[x] == x){
            return x;
        }
        else{
            int oldfa=fa[x];
            fa[x] = find(fa[x]);
            d[x] += d[oldfa];
            return fa[x];
        }
    }
    void merge(int x, int y, ll c)
    {
        int dx = find(x);
        int dy = find(y);
        if(dx != dy)
        {
            d[dx]=d[y]+c-d[x];
            fa[dx]=dy;
            sz[dy]+=sz[dx];
        }
    }
    bool judge(int x, int y)
    {
        return find(x) == find(y);
    }
    int size(int x)
    {
        return sz[find(x)];    
    }
};
```



## 1.2 树状数组

```cpp
template <class T>
struct Fenwick {
    int n;
    vector<T> a;
    Fenwick(int n = 0): n(n), a(n + 1) {}
    void add(int x, const T &v) {
        for (int i = x; i <= n; i += i & -i) a[i] = a[i] + v;
    }
    T sum(int x) {
        T ans=0;
        for (int i = x; i > 0; i -= i & -i) ans = ans + a[i];
        return ans;
    }
    T rangeSum(int l, int r) {
        return sum(r) - sum(l - 1);
    }
    int get_kth(int k) {
        int pos = 0;
        for (int i = __lg(n); i >= 0; --i) {
            int nxt = pos + (1 << i);
            if (nxt <= n && a[nxt] < k) {
                pos = nxt;
                k -= a[pos];
            }
        }
        return pos + 1;
    }
};
```

## 1.3 segtree

```cpp
struct SegTree {
    struct Info {
        ll val;
        Info operator+(const Info& x) const { return {max(val, x.val)}; }
        void operator+=(const Info& x) { *this = *this + x; }
    };
    int min_, max_;
    vector<Info> info;
    SegTree(int n): min_(1), max_(n), info(4 * n) {}
    SegTree(int l, int r): min_(l), max_(r), info(4 * (r - l + 1)) {}
private:
    void pull(int i) { info[i] = info[2 * i] + info[2 * i + 1]; }
    void set(int pos, Info v, int i, int l, int r) {
        if (l == r) {info[i] = v; return;}
        if (pos <= l + (r - l) / 2) set(pos, v, 2 * i, l, l + (r - l) / 2);
        else set(pos, v, 2 * i + 1, l + (r - l) / 2 + 1, r);
        pull(i);
    }
    void add(int pos, Info v, int i, int l, int r) {
        if (l == r) {info[i] += v; return;}
        if (pos <= l + (r - l) / 2) add(pos, v, 2 * i, l, l + (r - l) / 2);
        else add(pos, v, 2 * i + 1, l + (r - l) / 2 + 1, r);
        pull(i);
    }
    Info query(int x, int y, int i, int l, int r) {
        if (l >= x && r <= y) return info[i];
        if (y <= l + (r - l) / 2) return query(x, y, 2 * i, l, l + (r - l) / 2);
        if (x > l + (r - l) / 2) return query(x, y, 2 * i + 1, l + (r - l) / 2 + 1, r);
        return query(x, y, 2 * i, l, l + (r - l) / 2) + query(x, y, 2 * i + 1, l + (r - l) / 2 + 1, r);
    }
    int binarySearch(int i, int l, int r) {
        if (l == r) return l;
        if (info[2 * i].val <= 0) return binarySearch(2 * i, l, l + (r - l) / 2);
        return binarySearch(2 * i + 1, l + (r - l) / 2 + 1, r);
    }
public:
    void set(int i, const Info& v) { set(i, v, 1, min_, max_); }
    void add(int i, const Info& v) { add(i, v, 1, min_, max_); }
    Info query(int l, int r) { return query(l, r, 1, min_, max_); }
    int binarySearch() { return binarySearch(1, min_, max_); }
};
```

## 1.4 Lazysegtree

```cpp
struct LazySegTree {
	struct Info {
		ll val = 0;
		Info operator+(const Info& x) const { return {min(val, x.val)}; }
		void operator+=(const Info& x) { *this = *this + x; }
	};
	struct Lazy {
		ll val = 0;
		Lazy operator+(const Lazy& x) { return { val += x.val }; }
		void operator+=(const Lazy& x) { *this = *this + x; }
	};
	int min_, max_;
	vector<Info> info;
	vector<Lazy> tag;
	LazySegTree(int n): min_(1), max_(n), info(4 * n), tag(4 * n) {}
	LazySegTree(int l, int r): min_(l), max_(r), info(4 * (r - l + 1)), tag(4 * (r - l + 1)) {}
private:
	void operation(const Lazy& lazy, int i, int l, int r) {
		info[i].val += lazy.val;
		tag[i] += lazy;
	}
	void pull(int i) { info[i] = info[2 * i] + info[2 * i + 1]; }
	void push(int i, int l, int r) {
		operation(tag[i], 2 * i, l, (l + r) / 2);
		operation(tag[i], 2 * i + 1, (l + r) / 2 + 1, r);
		tag[i] = {};
	}
	void set(int pos, const Info& v, int i, int l, int r) {
		if (l == r) { info[i] = v; return; }
		push(i, l, r);
		if (pos <= (l + r) / 2) set(pos, v, 2 * i, l, (l + r) / 2);
		else set(pos, v, 2 * i + 1, (l + r) / 2 + 1, r);
		pull(i);
	}
	void rangeOperation(int x, int y, const Lazy& v, int i, int l, int r) {
		if (y < x || l > y || r < x) return;
		if (l >= x && r <= y) return operation(v, i, l, r);
		push(i, l, r);
		rangeOperation(x, y, v, 2 * i, l, (l + r) / 2);
		rangeOperation(x, y, v, 2 * i + 1, (l + r) / 2 + 1, r);
		pull(i);
	}
	Info query(int x, int y, int i, int l, int r) {
		if (l >= x && r <= y) return info[i];
		push(i, l, r);
		if (y <= (l + r) / 2) return query(x, y, 2 * i, l, (l + r) / 2);
		if (x > (l + r) / 2) return query(x, y, 2 * i + 1, (l + r) / 2 + 1, r);
		return query(x, y, 2 * i, l, (l + r) / 2) + query(x, y, 2 * i + 1, (l + r) / 2 + 1, r);
	}
public:
	void set(int i, const Info& v) { set(i, v, 1, min_, max_); }
	void rangeOperation(int l, int r, const Lazy& v) { rangeOperation(l, r, v, 1, min_, max_); }
	Info query(int l, int r) { return query(l, r, 1, min_, max_); }
	friend ostream& operator<<(ostream& os, LazySegTree& seg) {
		os << "seg[";
		for (int i = seg.min_; i <= seg.max_; i++) {
			os << seg.query(i, i).val << ",";
		}
		os << "]";
		return os;
	}
};
```

## 1.5 动态开点线段树

```cpp
struct SegmentTree {
	struct Info {
		ll sum = 0;
		ll cnt = 0;
		ll val = 0;
		Info(){}
		Info(ll v): sum(v), cnt(1) {}
		Info(ll sum, ll cnt, ll val): sum(sum), cnt(cnt), val(val) {}
		Info operator+(const Info& x) {
			return {sum + x.sum, cnt + x.cnt, val + x.val + x.sum * cnt - x.cnt * sum};
		}
		void operator+=(const Info& x) {
			*this = *this + x;
		}
	};
	struct Node {
		Info info;
		int ls = 0;
		int rs = 0;
		Node(){}
		Node(const Info& x): info(x) {}
	};
	ll min_, max_;
	vector<Node> node;

	SegmentTree(ll l, ll r): min_(l), max_(r), node(2) {}
	SegmentTree(): min_(0), max_(1e18), node(2) {}

private: //internal function part
	void pull(int i) {
		node[i].info = node[node[i].ls].info + node[node[i].rs].info;
	}
	void add(const ll& pos, const Info& val, int i, ll l, ll r) {
		if (l == r) { node[i].info += val; return; }
		ll m = l + (r - l) / 2;
		if (pos <= m) {
			if (node[i].ls == 0) {
				node[i].ls = node.size();
				node.emplace_back();
			}
			add(pos, val, node[i].ls, l, l + (r - l) / 2);
		} else {
			if (node[i].rs == 0) {
				node[i].rs = node.size();
				node.emplace_back();
			}
			add(pos, val, node[i].rs, l + (r - l) / 2 + 1, r);
		}
		pull(i);
	}

	Info query(const ll& x, const ll& y, int i, ll l, ll r) {
		ll m = l + (r - l) / 2;
		if (l >= x && r <= y) {
			return node[i].info;
		}
		if(y <= m){
			return query(x, y, node[i].ls, l, l + (r - l) / 2);
		} else if(x > m){
			return query(x, y, node[i].rs, l + (r - l) / 2 + 1, r);
		} else {
			return query(x, y, node[i].ls, l, l + (r - l) / 2) + query(x, y, node[i].rs, l + (r - l) / 2 + 1, r);
		}
	}

	ll kBiggest(int k, int i, ll l, ll r) {
		if (l == r) {
			return l;
		}
		if (node[i].rs == 0) {
			return kBiggest(k, node[i].ls, l, l + (r - l) / 2);
		}
		int d = node[node[i].rs].info.sum;
		if (k <= d) {
			return kBiggest(k, node[i].rs, l + (r - l) / 2 + 1, r);
		} else {
			return kBiggest(k - d, node[i].ls, l, l + (r - l) / 2);
		}
	}

public: // open interface part
	void print(ostream& os, int i, ll l, ll r) const {
		if (l == r) {
			os << l << ": " << node[i].info.sum << endl;
			return;
		}
		if (node[i].ls) {
			print(os, node[i].ls, l, l + (r - l) / 2);
		}
		if (node[i].rs) {
			print(os, node[i].rs, l + (r - l) / 2 + 1, r);
		}
	}
	void add(const ll& num) {
		add(num, 1, 1, min_, max_);
	}
	Info query(const ll& x, const ll& y) {
		return query(x, y, 1, min_, max_);
	}
	Info query() {
		return query(min_, max_, 1, min_, max_);
	}
	ll kBiggest(int k) {return kBiggest(k, 1, min_, max_);}
	friend ostream& operator<<(ostream& os, const SegmentTree& seg) {
		seg.print(os, 0, seg.min_, seg.max_);
		return os;
	}
};
```

## 1.6 可持久化线段树

```cpp
struct PST {
    struct Info {
        ll val = 0;
		Info operator+(const Info& x) {
            return {val + x.val};
		}
		void operator+=(const Info& x) {
            *this = *this + x;
		}
	};
	struct Node {
		Info info;
		int ls = 0;
		int rs = 0;
		Node operator+(const Info& x) {
            Node nd = *this;
			nd.info += x;
			return nd;
		}
		Node() {}
		Node(const Info& x) {
            info=x;
		}
	};
	int n = 0, min_ = 0, max_ = 1e9;
	vector<Node> node;
    
	PST(): node(1) {}
	PST(int n): min_(1), max_(n), node(1) {
        node.reserve(n*25);
    }
	PST(int l, int r): min_(l), max_(r), node(1) {}

private: //internal function part
    void pull(int i) {
        node[i].info = node[node[i].ls].info + node[node[i].rs].info;
	}
	int set(const int& pos, const Info& val, int old_version, int l, int r) {
        int cur = node.size();
		Node tmp = node[old_version];
        node.push_back(tmp);
		if (l == r) {
            node[cur].info = val;
			return cur;
		}
		int m = l + (r - l) / 2;
		if (pos <= m) {
            int ls = set(pos, val, node[old_version].ls, l, m);
			node[cur].ls = ls;
		} else {
            int rs = set(pos, val, node[old_version].rs, m + 1, r);
			node[cur].rs = rs;
		}
        pull(cur);
        return cur;
	}
	int add(const int& pos, const Info& val, int old_version, int l, int r) {
        int cur = node.size();
		Node tmp = node[old_version];
        node.push_back(tmp);
		if (l == r) {
            node[cur].info += val;
            return cur;
		}
		int m = l + (r - l) / 2;
		if (pos <= m) {
            int ls = add(pos, val, node[old_version].ls, l, m);
			node[cur].ls = ls;
		} else {
            int rs = add(pos, val, node[old_version].rs, m + 1, r);
			node[cur].rs = rs;
		}
        pull(cur);
		return cur;
    }
	Info query(const int& x, const int& y, int v, int l, int r) {
        int m = l + (r - l) / 2;
		if (l >= x && r <= y) {
            return node[v].info;
            debug(node[v].info.val);
		}
		if(y <= m){
            return query(x, y, node[v].ls, l, m);
		} else if(x > m){
            return query(x, y, node[v].rs, m + 1, r);
		} else {
            return query(x, y, node[v].ls, l, m) + query(x, y, node[v].rs, m + 1, r);
		}
	}
    int build(const vector<ll>& a, int l, int r){
        int cur = node.size();
        node.push_back(Node());
        if(l == r){
            node[cur].info = {a[l]};
            return cur;
        }
		int m = l +(r - l) / 2;
		int ls = build(a, l, m);
		int rs = build(a, m + 1, r);
		node[cur].ls = ls;
		node[cur].rs = rs;
		pull(cur);
		return cur;
    }
	void printVersion(int i, int l, int r) {
        if (l == r) {
			cout << '\t' << l << ": " << node[i].info.val << endl;
			return;
		}
		int m = l + (r - l) / 2;
		if (node[i].ls) {
            printVersion(node[i].ls, l, m);
		}
		if (node[i].rs) {
            printVersion(node[i].rs, m + 1, r);
		}
	}

public: // open interface part
	int build(const vector<ll>& a){
        return build(a, 1, a.size()-1);
	}
	int set(int i, const Info& v, int old_version) { // 返回值为对应版本号
		return set(i, v, old_version, min_, max_);
	}
	int add(int i, const Info& v, int old_version) { // 返回值为对应版本号
		return add(i, v, old_version, min_, max_);
	}
	Info query(int x, int y, int version) {
		return query(x, y, version, min_, max_);
	}
	void printVersion(int i) {
        cout << "print version " << i << ": " << endl;
		printVersion(i, min_, max_);
		cout << endl;
	}
};

```

## 1.7 ST表

```cpp
struct ST{
    int n;
    vector<vector<ll>> info;

    ST(const vector<ll>& vec){
        int n = vec.size() - 1;
        int maxlog = __lg(n) + 1;
        info.assign(maxlog + 1, vector<ll>(n + 1, 0));
        for(int i=1;i<=n;++i){
            info[0][i] = vec[i];
        }
        for(int j=1;j<=maxlog;++j){
            for(int i=1;i+(1LL<<j)-1<=n;++i){
                info[j][i]=max(info[j-1][i], info[j-1][i+(1LL<<(j-1))]);
            }
        }
    }

    ll query(int l, int r){
        int k = __lg(r-l+1);
        return max(info[k][l], info[k][r-(1LL<<k)+1]);
    }
};
```

## 1.8 笛卡尔树

```cpp
void solve(){
    ll n;
    cin>>n;
    vector<ll> p(n+1);
    fp(i,1,n) cin>>p[i];
    vector<ll> l(n+1);
    vector<ll> r(n+1);
    ll root=0;
    stack<ll> stk;
    
    fp(i,1,n){
        ll pre=0;
        while(stk.size() and p[i]<p[stk.top()]){
            pre=stk.top();
            stk.pop();
        }
        if(pre and stk.empty()){
            l[i]=pre;
            root=i;
        }
        else if(pre and stk.size()){
            r[stk.top()]=i;
            l[i]=pre;
        }
        else if(stk.size()){
            r[stk.top()]=i;
        }
        else root=i;
        stk.push(i);
    }

    ll ans1=0;
    ll ans2=0;
    fp(i,1,n){
        ans1^=i*(l[i]+1);
        ans2^=i*(r[i]+1);
    }
    cout<<ans1<<" "<<ans2;
}
```

## 1.9 FHQTreap

```cpp
struct FHQTreap{
    struct Node{
        int l,r;
        ll val,key;
        int size;
        Node(ll v,ll k=rng(),int sz=1){
            val=v;
            key=k;
            size=sz;
            l=r=0;
        }
    };
    int root=0;
    vector<Node> tr;
    
    FHQTreap(){
        tr.push_back(Node(0,0,0));
    }
    
    int newnode(ll val){
        tr.push_back(Node(val));
        return tr.size()-1;
    }
    void pushup(ll p){
        tr[p].size=tr[tr[p].l].size+tr[tr[p].r].size+1;
    }
    void split(int p,ll val,int &x,int &y){
        if(p==0) {
            x=y=0;
            return;
        }
        if(tr[p].val<=val){
            x=p;
            split(tr[p].r,val,tr[p].r,y);
        }
        else{
            y=p;
            split(tr[p].l,val,x,tr[p].l);
        }
        pushup(p);
    }
    int merge(int x,int y){
        if(!x or !y) return x+y;
        if(tr[x].key>tr[y].key){
            tr[x].r=merge(tr[x].r,y);
            pushup(x);
            return x;
        }
        else{
            tr[y].l=merge(x,tr[y].l);
            pushup(y);
            return y;
        }
    }
    void insert(ll v){
        int x,y;
        split(root,v,x,y);
        root=merge(merge(x,newnode(v)),y);
    }
    void del(ll v){
        int x,y,z;
        split(root,v,x,z);
        split(x,v-1,x,y);
        if(y) y=merge(tr[y].l,tr[y].r);
        root=merge(merge(x,y),z);
    }
    int getrank(ll v){
        int x,y;
        split(root,v-1,x,y);
        int res=tr[x].size+1;
        root=merge(x,y);
        return res;
    }
    ll getvalue(int rk){
        int p=root;
        while(p){
            if(tr[tr[p].l].size+1==rk) return tr[p].val;
            if(tr[tr[p].l].size>=rk) p=tr[p].l;
            else{
                rk-=(tr[tr[p].l].size+1);
                p=tr[p].r;
            }
        }
        return -1;
    }
    ll getpre(ll v){
        int x,y;
        split(root,v-1,x,y);
        int p=x;
        while(tr[p].r) p=tr[p].r;
        ll res=tr[p].val;
        root=merge(x,y);
        return res;
    }
    ll getnxt(ll v){
        int x,y;
        split(root,v,x,y);
        int p=y;
        while(tr[p].l) p=tr[p].l;
        ll res=tr[p].val;
        root=merge(x,y);
        return res;
    }
};
```

## 1.10 普通莫队

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
        while(curl<l){
            remove(curl);
            curl++;
        }
        while(curr<r){
            add(curr+1);
            curr++;
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



# 2. 图论

## 2.1 kruskal求最小生成树

```cpp
struct edge{
    ll x,y,z;
};

bool cmp(edge x,edge y){
    return x.z<y.z;
}

void solve(){
    ll n,m;
    cin>>n>>m;
    DSU dsu(n);
    vector<edge> edges(m+1);
    for(int i=1;i<=m;++i){
        ll x,y,z;
        cin>>x>>y>>z;
        edges[i].x=x;
        edges[i].y=y;
        edges[i].z=z;
    }
    sort(edges.begin()+1,edges.end(),cmp);
    ll cnt=0;
    ll ans=0;
    for(int i=1;i<=m;++i){
        if(cnt==n-1) break;
        ll x=edges[i].x;
        ll y=edges[i].y;
        ll z=edges[i].z;
        if(!dsu.judge(x,y)){
            cnt++;
            ans+=z;
            dsu.merge(x,y);
        }
    }
    if(cnt<n-1) {cout<<"orz";return;}
    cout<<ans;
}
```

## 2.1.1 重构树

```cpp
struct ex_kruskal
{
    int n,tot;
    vector<int> fa; //DSU的fa数组
    vector<ll> val;
    vector<int> lc,rc;
    vector<array<int, 22>> f;  //重构树的fa倍增数组
    vector<int> dep;
    explicit ex_kruskal(int n) : n(n),tot(n)
    {
        fa.assign(2*n + 1, 0);
        val.assign(2*n + 1, 0);
        lc.assign(2*n + 1, 0);
        rc.assign(2*n + 1, 0);
        f.assign(2*n + 1,array<int, 22>{});
        dep.assign(2*n + 1, 0);
        for (int i = 1; i <= 2*n-1; i++)
            fa[i] = i;
    }
    int find(int x)
    {
        if(fa[x] == x)
            return x;
        return fa[x] = find(fa[x]);
    }
    void merge(int x, int y, int w)
    {
        int dx = find(x);
        int dy = find(y);
        if(dx != dy)
        {
            fa[dx] = fa[dy] = ++tot;
            val[tot]=w;
            lc[tot]=dx;
            rc[tot]=dy;
        }
    }
    bool judge(int x, int y)
    {
        return find(x) == find(y);
    }
    void init(){
        auto dfs=[&](auto&&self,ll u,ll d) ->void {
            dep[u]=d;
            if(lc[u]){
                f[lc[u]][0]=u;
                self(self,lc[u],d+1);
            }
            if(rc[u]){
                f[rc[u]][0]=u;
                self(self,rc[u],d+1);
            }
        };
        for(int i=1;i<=tot;++i){
            if(fa[i]==i){
                f[i][0]=i;
                dfs(dfs,i,1);
            }
        }
        for(int j=1;j<=21;++j) for(int i=1;i<=2*n-1;++i)
            f[i][j]=f[f[i][j-1]][j-1];
    }
    int lca(int x, int y)
    {
        if(dep[x]<dep[y]) swap(x,y);
        while(dep[x] > dep[y])
            x = f[x][__lg(dep[x] - dep[y])];
        if(x==y) return x;
        for(int i=21;i>=0;--i){
            if(f[x][i]!=f[y][i]){
                x=f[x][i];
                y=f[y][i];
            }
        }
        return f[x][0];
    }

    ll query(int x, int y){
        if (!judge(x,y)) return -1;
        return val[lca(x,y)];
    }
};

void solve(){
    ll n,q;
    cin>>n>>q;
    vector<ll> a(n+1);
    fp(i,1,n) cin>>a[i];
    vector<vector<ll>> has(N+1);
    fp(i,1,n) has[a[i]].push_back(i);
    ex_kruskal tr(n);
    fq(i,N,1){
        for(int j=0;j<=(int)has[i].size()-2;++j){
            tr.merge(has[i][j],has[i][j+1],i);
        }
        ll pre=0;
        for(int j=i;j<=N;j+=i){
            if(has[j].size()){
                if(!pre) pre=has[j][0];
                tr.merge(pre,has[j][0],i);
            }
        }
    }
    tr.init();
    while(q--){
        ll x,y;
        cin>>x>>y;
        if(x==y) cout<<a[x]<<"\n";
        else cout<<tr.query(x,y)<<"\n";
    }
}
```



## 2.2 Prim求最小生成树

```cpp
void solve(){
    ll n,m;
    cin>>n>>m;
    vector<vector<pll>> G(n+1);
    while(m--){
        ll x,y,z;
        cin>>x>>y>>z;
        G[x].push_back({y,z});
        G[y].push_back({x,z});
    }
    vector<bool> vis(n+1);
    ll cnt=1;
    ll ans=0;
    vis[1]=1;
    priority_queue<pll,vector<pll>,greater<pll>> pq;
    for(auto [v,c]:G[1]){
        pq.push({c,v});
    }
    while(!pq.empty()){
        pll t=pq.top();
        pq.pop();
        ll c=t.fi;
        ll v=t.se;
        if(vis[v]) continue;
        vis[v]=1;
        ans+=c;
        cnt++;
        if(cnt==n) break;
        for(auto [v1,c1]:G[v]){
            if(vis[v1]) continue;
            pq.push({c1,v1});
        }
    }
    if(cnt==n) cout<<ans;
    else cout<<"orz";
}
```

## 2.3 dijkstra

```cpp
vector<ll> dijkstra(ll n,ll st){
    vector<ll> dis(n+1,INF); 
    bitset<N> vis;
    priority_queue<pll,vector<pll>,greater<pll>> pq;
    dis[st]=0;
    pq.push({0LL,st});
    while(!pq.empty()){
        ll u=pq.top().second;
        pq.pop();
        if(vis[u]) continue;
        vis[u]=1;
        for(auto [v,w]:G[u]){
            if(dis[u]+w<dis[v]){
                dis[v]=dis[u]+w;
                pq.push({dis[v],v});
            }
        }
    }
    return dis;
}
```

## 2.4 Floyd

```cpp
void solve(){
    ll n,m;
    cin>>n>>m;
    vector<vector<ll>> a(n+1,vector<ll>(n+1,INF));
    for(int i=1;i<=m;++i){
        ll u,v,w;
        cin>>u>>v>>w;
        a[u][v]=min(a[u][v],w);
        a[v][u]=min(a[v][u],w);
    }
    for(int i=1;i<=n;++i) a[i][i]=0;
    for(int k=1;k<=n;++k){
        for(int i=1;i<=n;++i){
            for(int j=1;j<=n;++j){
                a[i][j]=min(a[i][j],a[i][k]+a[k][j]);
            }
        }
    }
    for(int i=1;i<=n;++i){
        for(int j=1;j<=n;++j){
            cout<<a[i][j]<<" ";
        }
        cout<<"\n";
    }
}
```



## 2.5 匈牙利算法

```cpp
void solve(){
    ll n,m,e;
    cin>>n>>m>>e;
    vector<vector<ll>> G(n+1);
    fp(i,1,e){
        ll u,v;
        cin>>u>>v;
        G[u].push_back(v);
    }
    vector<bool> vis(m+1);
    vector<ll> match(m+1);
    auto find=[&](auto&&self,ll x) ->bool {
        for(auto y:G[x]){
            if(vis[y]) continue;
            vis[y]=1;
            if(match[y]==0 or self(self,match[y])){
                match[y]=x;
                return 1;
            }
        }
        return 0;
    };
    ll ans=0;
    fp(i,1,n) {
        vis.assign(m+1,0);
        if(find(find,i)) ans++;
    }
    cout<<ans; 
}
```

## 2.6 Tarjan求有向图强连通分量

```cpp
vector<vector<ll>> g(N);
stack<ll> stk;
bitset<N> instk;
vector<ll> dfn(N),low(N);
ll timer_,scc_cnt;
vector<ll> scc_id(N),scc_sz(N);

void tarjan(ll u){
    stk.push(u);
    low[u]=dfn[u]=++timer_;
    instk[u]=1;
    for(auto &v:g[u]){
        if(dfn[v]==0){
            tarjan(v);
            low[u]=min(low[u],low[v]);
        }
        else if(instk[v]){
            low[u]=min(low[u],dfn[v]);
        }
    }

    if(low[u]==dfn[u]){
        ++scc_cnt;
        while(1){
            ll x=stk.top();
            stk.pop();
            scc_id[x]=scc_cnt;
            instk[x]=0;
            scc_sz[scc_cnt]++;
            if(x==u) break;
        }
    }
}

void solve(){
    ll n,m;
    cin>>n>>m;
    for(int i=1;i<=m;++i){
        ll u,v;
        cin>>u>>v;
        g[u].push_back(v);
    }
    for(int i=1;i<=n;++i){
        if(!dfn[i]) tarjan(i);
    }
    cout<<scc_cnt<<"\n";
    vector<ll> call(n+1);
    vector<vector<ll>> comp(scc_cnt);
    for(int i=1;i<=n;++i){
        if(call[i]) continue;
        ll id=scc_id[i];
        for(int j=1;j<=n;++j){
            if(scc_id[j]==id){
                cout<<j<<" ";
                call[j]=1;
            }
        }
        cout<<"\n";
    }
}
```

## 2.7 缩点

```cpp
vector<vector<ll>> g(N);
vector<ll> w(N);

stack<ll> stk;
bitset<N> instk;
vector<ll> dfn(N),low(N),scc_id(N);
ll timer_,scc_cnt;

vector<vector<ll>> dag(N);
vector<ll> indeg(N);
vector<ll> outdeg(N);
vector<ll> scc_w(N);

vector<ll> dp(N,-1);


void tarjan(ll u){
    stk.push(u);
    low[u]=dfn[u]=++timer_;
    instk[u]=1;
    for(auto &v:g[u]){
        if(dfn[v]==0){
            tarjan(v);
            low[u]=min(low[u],low[v]);
        }
        else if(instk[v]){
            low[u]=min(low[u],dfn[v]);
        }
    }

    if(low[u]==dfn[u]){
        ++scc_cnt;
        while(1){
            ll x=stk.top();
            stk.pop();
            scc_id[x]=scc_cnt;
            instk[x]=0;
            scc_w[scc_cnt]+=w[x];
            if(x==u) break;
        }
    }
}

void solve(){
    ll n,m;
    cin>>n>>m;
    for(int i=1;i<=n;++i){
        cin>>w[i];
    }
    for(int i=1;i<=m;++i){
        ll u,v;
        cin>>u>>v;
        g[u].push_back(v);
    }

    for(int i=1;i<=n;++i){
        if(!dfn[i]) tarjan(i);
    }

    for(int i=1;i<=n;++i){
        for(auto &x:g[i]){
            ll u=scc_id[i];
            ll v=scc_id[x];
            if(u==v) continue;
            dag[u].push_back(v);
            indeg[v]++;
            outdeg[u]++;
        }
    }
    ll ans=0;
    queue<ll> q;
    for(int i=1;i<=scc_cnt;++i){
        dp[i]=scc_w[i];
        if(indeg[i]==0) q.push(i);
    }
    while(!q.empty()){
        ll u=q.front();
        q.pop();
        ans=max(ans,dp[u]);
        for(auto &v:dag[u]){
            if(dp[v]<dp[u]+scc_w[v]){
                dp[v]=dp[u]+scc_w[v];
            }
            indeg[v]--;
            if(indeg[v]==0) q.push(v);
        }
    }
    cout<<ans;
}

int main(){
    ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
    int _ =1;
    // cin>> _;
    while(_--)
        solve();
    return 0;
}
```

## 2.8 dinic

```cpp
struct Edge{
    int to, next;
    ll w;
}edges[M];

int head[N];
int cnt=-1;

void add_edge(int u, int v, ll w){
    edges[++cnt].to = v;
    edges[cnt].w = w;
    edges[cnt].next = head[u];
    head[u] = cnt;
}

int n,m,s,t;

int lv[N],cur[N]; 

bool bfs(){
    memset(lv, -1, sizeof(lv));
    memcpy(cur, head, sizeof(head));
    lv[s] = 0;
    queue<int> q;
    q.push(s);
    while(q.size()){
        int p=q.front();
        q.pop();
        for(int eg=head[p]; eg != -1; eg=edges[eg].next){
            int to = edges[eg].to;
            ll vol = edges[eg].w;
            if(vol > 0 and lv[to] == -1){
                lv[to] = lv[p] + 1;
                q.push(to);
            }
        }
    }
    return (lv[t] != -1);
}

ll dfs(int p = s, ll flow = INF){
    if(p == t) return flow;
    ll rmn = flow;
    for(int eg=cur[p]; (eg!=-1) and rmn; eg = edges[eg].next){
        cur[p] = eg;
        int to = edges[eg].to;
        ll vol = edges[eg].w;
        if(vol and lv[to] == lv[p] + 1){
            ll c = dfs(to, min(vol, rmn));
            rmn -= c;
            edges[eg].w -= c;
            edges[eg ^ 1].w += c;
        }
    }
    return flow - rmn;
}

ll dinic(){
    ll ans=0;
    while(bfs()){
        ans += dfs();
    }
    return ans;
}

void solve(){
    cin>>n>>m>>s>>t;
    memset(head, -1, sizeof(head));
    fp(i,1,m){
        ll u,v,w;
        cin>>u>>v>>w;
        add_edge(u,v,w);
        add_edge(v,u,0);
    }
    cout<<dinic();
}
```

## 2.9 欧拉路径

欧拉路存在的充要条件如下：

首先，图是连通的。

对于无向图：

（1）欧拉路径存在条件：有且仅有两个点，与其相连的边数为奇数，其他点相连边数皆为偶数；或所有点皆为偶数边点。

对于两个奇数点，一个为起点，一个为终点。

（2）欧拉回路存在条件：如果存在这样一个欧拉路，其所有的点相连边数都为偶数，那说明它是欧拉回路。

对于有向图：

（1）欧拉路径存在条件：除去起点和终点，所有点的出度与入度相等。起点出度比入度大1，终点入度比出度大1。

（2）欧拉回路存在条件：若起点终点出入度也相同，则为欧拉回路。

### 2.9.1 无向图欧拉路径

```cpp
void solve(){
    ll n,m;
    cin>>n>>m;
    vector<vector<pll>> G(n+1);
    vector<ll> cur(n+1);
    vector<ll> deg(n+1);
    vector<bool> vis(m+1);
    ll tot=-1;
    DSU dsu(n);
    fp(i,1,m){
        ll u,v;
        cin>>u>>v;
        G[u].push_back({v,++tot});
        G[v].push_back({u,tot});
        dsu.merge(u,v);
        deg[u]++;
        deg[v]++;
    }
    ll cnt=0;
    ll st=0;
    fp(i,1,n){
        if(deg[i]%2!=0){
            cnt++;
            if(st==0) st=i;
        }
    }
    if(cnt!=0 and cnt!=2){
        cout<<"No\n";
        return;
    }
    if(st==0){
        fp(i,1,n) if(deg[i]!=0) {st=i;break;}
    }
    ll root=-1;
    fp(i,1,n){
        if(deg[i]==0) continue;
        ll tmp=dsu.find(i);
        if(root==-1) root=tmp;
        if(tmp!=root){
            cout<<"No\n";
            return;
        }
    }
    fp(i,1,n) sort(G[i].begin(),G[i].end());
    vector<ll> path;
    auto dfs=[&](auto&&self,ll u) ->void {
        for(auto &i=cur[u];i<G[u].size();){
            auto [v,id]=G[u][i];
            if(vis[id]){
                i++;
                continue;
            }
            vis[id]=1;
            i++;
            self(self,v);
        }
        path.push_back(u);
    };

    dfs(dfs,st);
    for(auto it=path.rbegin();it!=path.rend();it++){
        cout<<*it<<"\n";
    }
}
```

### 2.9.2 有向图欧拉路径

```cpp
void solve(){
    ll n,m;
    cin>>n>>m;
    vector<vector<pll>> G(n+1);
    vector<ll> cur(n+1);
    vector<ll> in(n+1);
    vector<ll> out(n+1);
    vector<bool> vis(m+1);
    ll tot=-1;
    DSU dsu(n);
    fp(i,1,m){
        ll u,v;
        cin>>u>>v;
        G[u].push_back({v,++tot});
        dsu.merge(u,v);
        in[v]++;
        out[u]++;
    }
    int flag1=0; // out-in=1
    int flag2=0; // in-out=1
    int st=1;
    fp(i,1,n){
        if(abs(in[i]-out[i])>=2){
            cout<<"No\n";
            return;
        }
        if(out[i]-in[i]==1) {flag1++;st=i;}
        if(in[i]-out[i]==1) flag2++;
    }
    if(flag1+flag2!=0 and (flag1!=1 or flag2!=1)){
        cout<<"No\n";
        return;
    }
    ll root=-1;
    fp(i,1,n){
        if(in[i]==0 and out[i]==0) continue;
        ll tmp=dsu.find(i);
        if(root==-1) root=tmp;
        if(tmp!=root){
            cout<<"No\n";
            return;
        }
    }
    fp(i,1,n) sort(G[i].begin(),G[i].end());
    vector<ll> path;
    auto dfs=[&](auto&&self,ll u) ->void {
        for(auto &i=cur[u];i<G[u].size();){
            auto [v,id]=G[u][i];
            if(vis[id]){
                i++;
                continue;
            }
            vis[id]=1;
            i++;
            self(self,v);
        }
        path.push_back(u);
    };

    dfs(dfs,st);
    for(auto it=path.rbegin();it!=path.rend();it++){
        cout<<*it<<" ";
    }
}
```

# 3.  树

## 3.1 lca(倍增)

```cpp
void solve(){
    ll n,m,s;
    cin>>n>>m>>s;
    vector<vector<ll>> G(n+1);
    fp(i,1,n-1){
        ll l,r;
        cin>>l>>r;
        G[l].push_back(r);
        G[r].push_back(l);
    }
    
    vector<vector<ll>> fa(n+1,vector<ll>(21,0));
    vector<ll> dep(n+1);

    auto dfs=[&](auto&& self,ll u,ll d) ->void {
        dep[u]=d;
        for(auto &v:G[u]){
            if(v==fa[u][0]) continue;
            fa[v][0]=u;
            self(self,v,d+1);
        }
    };

    dfs(dfs,1,1);

    fp(j,1,20) fp(i,1,n) fa[i][j]=fa[fa[i][j-1]][j-1];

    auto lca=[&](ll a,ll b) ->ll {
        if(a==b) return a;
        if(dep[a]<dep[b]) swap(a,b);
        ll d=dep[a]-dep[b];
        fp(i,0,20){
            if(d&(1LL<<i)) a=fa[a][i];
        }
        if(a==b) return a;
        fq(i,20,0){
            if(fa[a][i]!=fa[b][i]){
                a=fa[a][i];
                b=fa[b][i];
            }
        }
        return fa[a][0];
    };

    fp(i,1,m){
        ll a,b;
        cin>>a>>b;
        cout<<lca(a,b)<<"\n";
    }
}
```

## 3.2 lca(tarjan)

```cpp
void solve(){
    ll n,m,s;
    cin>>n>>m>>s;
    vector<vector<ll>> G(n+1);
    fp(i,1,n-1){
        ll x,y;
        cin>>x>>y;
        G[x].push_back(y);
        G[y].push_back(x);
    }
    vector<bool> vis(m+1);
    vector<ll> ans(m+1);
    vector<ll> fa(n+1);
    DSU dsu(n+1);
    vector<vector<array<ll,2>>> qry(m+1);
    fp(i,1,m){
        ll a,b;
        cin>>a>>b;
        qry[a].push_back({i,b});
        qry[b].push_back({i,a});
    }

    auto dfs=[&](auto&& self, ll u) ->void {
        for(auto &v:G[u]){
            if(v==fa[u]) continue;
            fa[v]=u;
            self(self,v);
        }
        for(auto &[id,x]:qry[u]){
            if(vis[id]) ans[id]=dsu.find(x);
            else vis[id]=1;
        }
        dsu.merge(u,fa[u]);
    };

    dfs(dfs,s);

    fp(i,1,m) cout<<ans[i]<<"\n";
}
```

## 3.3 重链剖分

```cpp
void solve(){
    ll n,q,r;
    cin>>n>>q>>r;
    // cin>>n>>q; r=1;
    vector<ll> val(n+1);
    vector<ll> dep(n+1);
    vector<ll> fa(n+1);
    vector<ll> sz(n+1);
    vector<ll> id(n+1);
    ll _cnt=0;
    vector<ll> son(n+1);
    vector<ll> top(n+1);
    // LazySegTree seg(n);
    // fp(i,1,n) cin>>val[i];
    vector<vector<ll>> G(n+1);
    fp(i,1,n-1){
        ll x,y;
        cin>>x>>y;
        G[x].push_back(y);
        G[y].push_back(x);
    }
    
    auto dfs1=[&](auto&& self,ll u,ll f,ll d) ->void {
        dep[u]=d;
        fa[u]=f;
        sz[u]=1;
        ll maxv=0;
        ll maxid=0;
        for(auto &v:G[u]){
            if(v==f) continue;
            self(self,v,u,d+1);
            sz[u]+=sz[v];
            if(sz[v]>maxv) {
                maxv=sz[v];
                maxid=v;
            }
        }
        son[u]=maxid;
    };
    
    dfs1(dfs1,r,0,1);
    
    auto dfs2=[&](auto&& self,ll u,ll t) ->void {
        top[u]=t;
        id[u]=++_cnt;
        if(son[u]>0){
            self(self,son[u],t);
        }
        for(auto &v:G[u]){
            if(v==fa[u] or v==son[u]) continue;
            self(self,v,v);
        }
    };

    dfs2(dfs2,r,r);
    
    fp(i,1,n) seg.set(id[i],{val[i]%MOD});
    
    auto pathadd=[&](ll x,ll y,ll v) {
        while(top[x]!=top[y]){
            if(dep[top[x]]<dep[top[y]]) swap(x,y);
            seg.rangeOperation(id[top[x]],id[x],{v});
            x=fa[top[x]];
        }
        if(dep[x]<dep[y]) swap(x,y);
        seg.rangeOperation(id[y],id[x],{v});
    };

    auto pathquery=[&](ll x,ll y){
        ll ans=0;
        while(top[x]!=top[y]){
            if(dep[top[x]]<dep[top[y]]) swap(x,y);
            ans+=seg.query(id[top[x]],id[x]).val;
            ans%=MOD;
            x=fa[top[x]];
        }
        if(dep[x]<dep[y]) swap(x,y);
        ans+=seg.query(id[y],id[x]).val;
        ans%=MOD;
        return ans;
    };

    auto subtreeadd=[&](ll x,ll v){
        seg.rangeOperation(id[x],id[x]+sz[x]-1,{v});
    };

    auto subtreequery=[&](ll x) ->ll {
        return seg.query(id[x],id[x]+sz[x]-1).val;
    };

    auto lca=[&](ll x,ll y) ->ll {
        while(top[x]!=top[y]){
            if(dep[top[x]]<dep[top[y]]) swap(x,y);
            x=fa[top[x]];
        }
        if(dep[x]<dep[y]) return x;
        else return y;
    };
}
```

## 3.4 树上启发式合并

```cpp
void solve(){
    int n;
    cin>>n;
    vector<vector<int>> G(n+1);
    fp(i,1,n-1){
        int u,v;
        cin>>u>>v;
        G[u].push_back(v);
        G[v].push_back(u);
    }
    vector<ll> c(n+1);
    vector<ll> fa(n+1);
    vector<ll> mxson(n+1);
    vector<ll> sz(n+1);
    fp(i,1,n) cin>>c[i];

    vector<ll> cnt(n+1);
    vector<ll> res(n+1);
    ll ans=0;

    auto dfs1=[&](auto&&self,ll u) ->void {
        int maxsz=0;
        sz[u]=1;
        for(auto &v:G[u]){
            if(v==fa[u]) continue;
            fa[v]=u;
            self(self,v);
            sz[u]+=sz[v];
            if(sz[v]>maxsz){
                maxsz=sz[v];
                mxson[u]=v;
            }
        }
    };

    dfs1(dfs1,1);

    int skip=0;

    auto update=[&](auto&& self,int u,int val) -> void {
        if(val==1){
            if(cnt[c[u]]==0) ans++;
            cnt[c[u]]++;
            for(auto &v:G[u]){
                if(v!=skip and v!=fa[u]) self(self,v,val);
            }
        }
        else{
            if(cnt[c[u]]==1) ans--;
            cnt[c[u]]--;
            for(auto &v:G[u]){
                if(v!=skip and v!=fa[u]) self(self,v,val);
            }
        }
    };

    auto dfs2=[&](auto&& self,ll u,bool keep) ->void {
        for(auto &v:G[u]){
            if(v==mxson[u] or v==fa[u]) continue;
            self(self,v,0);
        }
        if(mxson[u]) self(self,mxson[u],1);

        skip=mxson[u];
        update(update,u,1);
        res[u]=ans;
        
        if(!keep) {
            skip=0;
            update(update,u,-1);
        }
    };

    dfs2(dfs2,1,0);

    int m;
    cin>>m;

    while(m--){
        int x;
        cin>>x;
        cout<<res[x]<<"\n";
    }
}
```

## 3.5 树哈希

主要用来判断树是否同构。

```cpp
ull mask(ull x) {
    x ^= x << 13;
    x ^= x >> 7;
    x ^= x << 17;
    return x;
}
void solve(){
    vector<ull> h(n+1); //哈希值
    auto get_hash=[&](auto&& self,ll u) ->void {
        h[u] = 1;
        for (auto v : G[u]) {
            self(self,v);
            h[u] += mask(h[v]);
        }
    };

    get_hash(get_hash,0);
```

# 4. 数学

## 4.1 快速幂和组合数

```cpp
vector<ll> fact(N+5);
vector<ll> facinv(N+5);

ll qpow(ll a,ll b,ll m=MOD){
    ll res=1;
    while(b){
        if(b%2) res=res*a%m;
        b>>=1;
        a=a*a%m;
    }
    return res;
}

void fac(){    //记得调用一下预处理
    fact[0]=1;
    for(int i=1;i<=N;++i){
        fact[i]=fact[i-1]*i%MOD;
    }
    facinv[N]=qpow(fact[N],MOD-2);
    for(int i=N-1;i>=0;--i){
        facinv[i]=facinv[i+1]*(i+1)%MOD;
    }
}

ll com(ll a,ll b){
    if(b>a or b<0) return 0;
    return fact[a]*facinv[a-b]%MOD*facinv[b]%MOD;
}

ll arr(ll a,ll b){
    if(b>a or b<0) return 0;
    return fact[a]*facinv[a-b]%MOD;
}
```

## 4.2 线性筛

```cpp
// 只筛素数
bitset<N> vis;
vector<ll> prime;

void sieve(){
    for(int i=2;i<=N-1;++i){
        if(!vis[i]) prime.push_back(i);
        for(int j=1;i*prime[j-1]<=N-1;++j){
            vis[i*prime[j-1]]=1;
            if(i%prime[j-1]==0) break;
        }
    }
}

// 同时找每个数的最小素因子
vector<ll> vis(N);
vector<ll> prime;

void sieve(){
    for(int i=2;i<=N-1;++i){
        if(!vis[i]) {
            prime.push_back(i);
            vis[i]=i;
        }
        for(int j=1;i*prime[j-1]<=N-1;++j){
            vis[i*prime[j-1]]=prime[j-1];
            if(i%prime[j-1]==0) break;
        }
    }
}
```

## 4.3 exgcd

```cpp
ll exgcd(ll a,ll b,ll &x,ll &y){
    if(b==0){
        x=1;
        y=0;
        return a;
    }
    ll d=exgcd(b,a%b,y,x);
    y-=(a/b)*x;
    return d;
}

void solve(){
    ll a,b,c,x,y;
    cin>>a>>b>>c;
    ll d=exgcd(a,b,x,y);
    if(c%d){
        cout<<-1<<"\n";
        return;
    }
    x=x*(c/d);
    y=y*(c/d);
    ll flag=0;
    ll minx=INF,miny=INF;
    ll maxx=-INF,maxy=-INF;
    ll dx=b/d,dy=a/d;
    ll x1=x,y1=y;
    if(x1<=0) {
        ll t=(0-x1)/dx+1;
        x1+=t*dx;
        y1-=t*dy;
    }
    else{
        ll t=(x1-1)/dx;
        x1 -= t*dx;
        y1 += t*dy;
    }
    if(y1>0) flag=1;
    minx=x1;
    maxy=y1;
    ll x2=x,y2=y;
    if(y2<=0) {
        ll t=(0-y2)/dy+1;
        y2+=t*dy;
        x2-=t*dx;
    }
    else{
        ll t=(y2-1)/dy;
        y2 -= t*dy;
        x2 += t*dx;
    }
    if(x2>0) flag=1;
    miny=y2;
    maxx=x2;
    if(flag) cout<<(maxx-minx)/dx+1<<" "<<minx<<" "<<miny<<" "<<maxx<<" "<<maxy<<"\n";
    else cout<<minx<<" "<<miny<<"\n";
}
```

## 4.4 线性求逆

```cpp
vector<ll> inv(N+1);

void inv_init(){
    inv[1]=1;
    for(int i=2;i<=N;++i){
        inv[i]=(MOD-MOD/i)*inv[MOD%i]%MOD;
    }
}
```

## 4.5 线性基

```cpp
struct XorLinearBasis{
    int n;
    vector<ll> base;
    vector<ll> mask;
    vector<int> id;

    XorLinearBasis(int n=63):n(n) {
        base.assign(n+1,0);
        mask.assign(n+1,0);
        id.assign(n+1,0);
    }

    bool insert(ll val,int idx){
        ll cur=0;
        for(int i=n;i>=0;--i){
            if(!((1LL<<i)&val)) continue;
            if(!base[i]){
                base[i]=val;
                cur^=(1LL<<i);
                mask[i]=cur;
                id[i]=idx;
                return 1;
            }
            val ^= base[i];
            cur ^= mask[i];
        }
        return 0;
    }

    //bool返回是否可行，如果可以，返回要选的数
    pair<bool,vector<ll>> check(ll x){
        ll ans=0;
        for(int i=n;i>=0;--i){
            if(x&(1LL<<i)){
                ans^=mask[i];
                x^=base[i];
            }
        }
        if(x) return {0,{0}};
        vector<ll> res;
        for(int i=0;i<=n;++i){
            if(ans&(1LL<<i)) res.push_back(id[i]);
        }
        return {1,res};
    }

    ll maxsum(){
        ll ans=0;
        for(int i=n;i>=0;--i){
            if(ans&(1LL<<i)) continue;
            ans ^= base[i];
        }
        return ans;
    }
    
};
```

## 4.6 矩阵类

```cpp
struct Mat{
    ll n;
    vector<vector<ll>> d;
    Mat(ll _n):n(_n) {
        d.assign(n+1,vector<ll>(n+1,0));
    }
    static Mat I(ll n){
        Mat res(n);
        for(int i=1;i<=n;++i){
            res.d[i][i]=1;
        }
        return res;
    }
    Mat operator+ (const Mat& x) const{
        Mat res(n);
        for(int i=1;i<=n;++i){
            for(int j=1;j<=n;++j){
                res.d[i][j]=(d[i][j]+x.d[i][j])%MOD;
            }
        }
        return res;
    }
    Mat& operator+= (const Mat& x){
        for(int i=1;i<=n;++i){
            for(int j=1;j<=n;++j){
                d[i][j]=(d[i][j]+x.d[i][j])%MOD;
            }
        }
        return *this;
    }
    Mat operator* (const Mat& x) const{
        Mat res(n);
        for(int i=1;i<=n;++i){
            for(int j=1;j<=n;++j){
                for(int k=1;k<=n;++k){
                    res.d[i][j]+=d[i][k]*x.d[k][j]%MOD;
                    res.d[i][j]%=MOD;
                }
            }
        }
        return res;
    }
    static Mat qpow(Mat A,ll b){
        ll n=A.n;
        Mat res = I(n);
        while(b){
            if(b&1) res=res*A;
            A=A*A;
            b>>=1;
        }
        return res;
    }
    // 返回 A^m 和 S(m-1)=A^0+A^1+A^2+…+A^m-1
    static pair<Mat,Mat> powsum(const Mat& A,ll m){  
        ll n=A.n;
        if(m==0) return {I(n),Mat(n)};
        if(m==1) return {A,I(n)};
        if(m&1){
            auto [P,S]=powsum(A,m-1);
            return {P*A,S+P};
        }
        else{
            auto [P,S]=powsum(A,m/2);
            return {P*P,S*P+S};
        }
    }
};
```

## 4.7 FFT

```cpp
struct Complex
{
    double x, y;
    Complex(double _x = 0.0, double _y = 0.0) : x(_x), y(_y) {}
    Complex operator+(const Complex &b) const { return Complex(x + b.x, y + b.y); }
    Complex operator-(const Complex &b) const { return Complex(x - b.x, y - b.y); }
    Complex operator*(const Complex &b) const { return Complex(x * b.x - y * b.y, x * b.y + y * b.x); }
};

class FFT
{
private:
    const double PI = acos(-1.0);
    vector<int> rev;
    int limit;

    void init(int n)
    {
        limit = 1;
        int l = 0;
        while (limit <= n)
        {
            limit <<= 1;
            l++;
        }
        rev.assign(limit, 0);
        for (int i = 0; i < limit; i++)
        {
            rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (l - 1));
        }
    }

    // type = 1 为 FFT，type = -1 为 IFFT
    void transform(vector<Complex> &A, int type)
    {
        for (int i = 0; i < limit; i++)
        {
            if (i < rev[i])
                swap(A[i], A[rev[i]]);
        }
        for (int mid = 1; mid < limit; mid <<= 1)
        {
            Complex Wn(cos(PI / mid), type * sin(PI / mid));
            int R = mid << 1;
            for (int j = 0; j < limit; j += R)
            {
                Complex w(1, 0);
                for (int k = 0; k < mid; ++k)
                {
                    Complex x = A[j + k], y = w * A[j + k + mid];
                    A[j + k] = x + y;
                    A[j + k + mid] = x - y;
                    w = w * Wn;
                }
            }
        }
        if (type == -1)
        {
            for (int i = 0; i < limit; ++i)
                A[i].x /= limit;
        }
    }

public:
    vector<ll> multiply(const vector<ll> &a, const vector<ll> &b)
    {
        if (a.empty() || b.empty())
            return {};
        int n = a.size() - 1, m = b.size() - 1;
        init(n + m);

        vector<Complex> A(limit), B(limit);
        for (int i = 0; i <= n; ++i)
            A[i] = Complex(a[i], 0);
        for (int i = 0; i <= m; ++i)
            B[i] = Complex(b[i], 0);

        transform(A, 1);
        transform(B, 1);

        for (int i = 0; i < limit; i++)
            A[i] = A[i] * B[i];

        transform(A, -1);

        vector<ll> res(n + m + 1);
        for (int i = 0; i <= n + m; ++i)
        {
            res[i] = (ll)(A[i].x + 0.5);
        }
        return res;
    }
};

void solve()
{
    FFT fft;
    ll n, m;
    cin >> n >> m;
    vector<ll> f(n + 1);
    vector<ll> g(m + 1);
    fp(i, 0, n) cin >> f[i];
    fp(i, 0, m) cin >> g[i];
    vector<ll> res = fft.multiply(f, g);
    fp(i, 0, n + m) cout << res[i] << " ";
}
```

## 4.8 卡特兰数

正方形  $\frac{1}{n+1} \binom{2n}{n} = \binom{2n}{n} - \binom{2n}{n-1}$

矩形($m \ge n$)  $\frac{m-n+1}{m+1} \binom{m+n}{n} = \binom{m+n}{m} - \binom{m+n}{m+1}$

## 4.9 MillerRabin素性检验

检验一个数是否是素数，复杂度$O(7*log(n))$。

```cpp
bool MRtest(ll n){
    if(n<3 or n%2==0) return n==2;
    ll u=n-1,t=0;
    while(u%2==0) u>>=1,t++;
    vector<ll> base={2,325,9375,28178,450775,9780504,1795265022};
    for(auto a:base){
        if(n<=a) break;
        lll v=qpow(a,u,n);
        if(v==1 or v==n-1) continue;
        bool flag=0;
        for(int j=1;j<t;++j){
            v=v*v%n;
            if(v==n-1){
                flag=1;
                break;
            }
        }
        if(!flag) return 0;
    }
    return 1;
}

void solve(){
    ll n;
    cin>>n;
    bool flag=MRtest(n);
    if(flag) cout<<"Yes\n";
    else cout<<"No\n";
}
```

## 4.10 CRT

```cpp
ll mul(ll a,ll b,ll mod){   //高精度乘法
    string x=to_string(a);
    ll res=0;
    for(int i=0;i<=x.size()-1;++i){
        res*=10;
        res+=(x[i]-'0')*b;
        res%=mod;
    }
    return res;
}

void solve(){
    ll n;
    cin>>n;
    vector<ll> a(n+1);  // 模数
    vector<ll> b(n+1);  // 余数
    vector<ll> m(n+1);
    vector<ll> inv(n+1);
    vector<ll> c(n+1);
    ll pro=1;
    for(int i=1;i<=n;++i) {cin>>a[i]>>b[i];pro*=a[i];}
    for(int i=1;i<=n;++i) {
        m[i]=(pro/a[i]);
        ll x,y;
        exgcd(m[i],a[i],x,y);
        inv[i]=(x+a[i])%a[i];
        c[i]=m[i]*inv[i];
    }
    ll ans=0;
    for(int i=1;i<=n;++i){
        ans=(ans+mul(b[i],c[i],pro)+pro)%pro;
    }
    cout<<ans;
}
```

# 5. 字符串

## 5.1 Hash

```cpp
struct custom_hash {
    static ull splitmix64(ull x) {
        x += 0x9e3779b97f4a7c15;
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9;
        x = (x ^ (x >> 27)) * 0x94d049bb133111eb;
        return x ^ (x >> 31);
    }
    size_t operator()(ull x) const {
        static const ull FIXED_RANDOM = chrono::steady_clock::now().time_since_epoch().count();
        return splitmix64(x + FIXED_RANDOM);
    }   
};

struct Hash {
    static inline ll MOD1 = 1e9 + 7, MOD2 = 1e9 + 9;
    static inline ll B1 = 0, B2 = 0;
    static inline vector<ll> p1 = {1}, p2 = {1};

    static void init(int mx_len) {
        if (B1 == 0) {
            mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());
            B1 = uniform_int_distribution<ll>(1e5, 1e9 - 10)(rng) | 1; // 强制奇数 Base
            B2 = uniform_int_distribution<ll>(1e5, 1e9 - 10)(rng) | 1;
        }
        while (p1.size() <= mx_len) {
            p1.push_back(p1.back() * B1 % MOD1);
            p2.push_back(p2.back() * B2 % MOD2);
        }
    }

    vector<ll> h1, h2;
    
    Hash(){}

    //传1-base的字符串
    Hash(const string& s) {
        int n = s.size();
        init(n);
        h1.resize(n + 1, 0); h2.resize(n + 1, 0);
        for (int i = 0; i < n; i++) {
            h1[i + 1] = (h1[i] * B1 + s[i]) % MOD1;
            h2[i + 1] = (h2[i] * B2 + s[i]) % MOD2;
        }
    }

    // 查询子串 [l, r] 的哈希值，0-indexed 闭区间
    ull get(int l, int r) const {
        ll r1 = (h1[r + 1] - h1[l] * p1[r - l + 1]) % MOD1;
        ll r2 = (h2[r + 1] - h2[l] * p2[r - l + 1]) % MOD2;
        if (r1 < 0) r1 += MOD1;
        if (r2 < 0) r2 += MOD2;
        return ((ull)r1 << 32) | r2; // 高32位放hash1，低32位放hash2
    }
};

void solve(){
    ll n;
    cin>>n;
    vector<string> s(n+1);
    fp(i,1,n) cin>>s[i];
    fp(i,1,n) s[i]=" "+s[i];
    vector<Hash> hash(n+1);
    fp(i,1,n) hash[i]=Hash(s[i]);
    vector<ll> l(n+1);
    fp(i,1,n) l[i]=(int)s[i].size()-1;
    ll ans=0;
    gp_hash_table<ull, ll, custom_hash> cnt;
    fp(i,1,n){
        fq(j,l[i],1){
            ans+=cnt[hash[i].get(j,l[i])];
        }
        fp(j,1,l[i]){
            cnt[hash[i].get(1,j)]++;
        }
    }
}
```

## 5.2 KMP

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
```

## 5.3 Manacher

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

## 6.4 Trie

```cpp
struct Trie{
    static constexpr int SIZE = 62;
    struct Node{
        int ch[SIZE] = {0};
        int pass=0;
        int end=0;
        Node() = default;
    };
    vector<Node> nodes;
    Trie(){
        nodes.reserve(200000);
        nodes.emplace_back();
    }

    int index(const char& ch) const {
        if(ch >='0' and ch<='9'){
            return ch-'0';
        }
        if(ch>='a' and ch<='z'){
            return ch-'a'+10;
        }
        if(ch>='A' and ch<='Z'){
            return ch-'A'+36;
        }
    }

    void insert(const string& s){
        int cur=0;
        for(char ch:s){
            int idx = index(ch);
            if(nodes[cur].ch[idx]==0){
                nodes[cur].ch[idx]=nodes.size();
                nodes.emplace_back();
            }
            cur=nodes[cur].ch[idx];
            nodes[cur].pass++;
        }
        nodes[cur].end++;
    }

    // 精确查找一个单词是否存在
    bool find(const string& s){
        int cur=0;
        for(char ch:s){
            int idx = index(ch);
            if(nodes[cur].ch[idx]==0){
                return 0;
            }
            cur=nodes[cur].ch[idx];
        }
        return nodes[cur].end > 0;
    }

    // 查找是否存在以指定prefix为前缀的单词
    int start(const string& s){
        int cur=0;
        for(char ch:s){
            int idx = index(ch);
            if(nodes[cur].ch[idx]==0){
                return 0;
            }
            cur=nodes[cur].ch[idx];
        }
        return nodes[cur].pass;
    }
};

void solve(){
    ll n,q;
    cin>>n>>q;
    Trie trie;
    while(n--){
        string s;
        cin>>s;
        trie.insert(s);
    }
    while(q--){
        string s;
        cin>>s;
        cout<<trie.start(s)<<"\n";
    }
}
```

## 6.5 ACAutomation

```cpp
struct ACAutomation{
    static constexpr int SIZE = 26;
    struct Node{
        int ch[SIZE] = {0};
        int fail=0;
        int count=0; //文本串经过该节点的次数
        int end=0;
        int match=0;
        vector<int> has;
        Node() = default;
    };
    vector<Node> nodes;
    vector<int> topo;
    int tot=0;

    ACAutomation(){
        nodes.reserve(200000);
        nodes.emplace_back();
    }

    int index(const char& ch) const {
        if(ch>='a' and ch<='z'){
            return ch-'a';
        }
    }

    void insert(const string& s){
        int cur=0;
        for(char ch:s){
            int idx = index(ch);
            if(nodes[cur].ch[idx]==0){
                nodes[cur].ch[idx]=nodes.size();
                nodes.emplace_back();
            }
            cur=nodes[cur].ch[idx];
        }
        nodes[cur].end++;
        nodes[cur].has.push_back(++tot);
    }

    void build(){
        queue<int> q;
        for(int i=0;i<SIZE;++i){
            if(nodes[0].ch[i]){
                q.push(nodes[0].ch[i]);
            }
        }
        while(q.size()){
            int u=q.front();
            topo.push_back(u);
            q.pop();
            nodes[u].match=nodes[u].end+nodes[nodes[u].fail].match;
            for(int i=0;i<SIZE;++i){
                if(nodes[u].ch[i]){
                    nodes[nodes[u].ch[i]].fail=nodes[nodes[u].fail].ch[i];
                    q.push(nodes[u].ch[i]);
                }
                else{
                    nodes[u].ch[i]=nodes[nodes[u].fail].ch[i];
                }
            }
        }
    }

    void query(const string& s){
        int cur=0;
        for(char ch:s){
            int idx=index(ch);
            cur=nodes[cur].ch[idx];
            nodes[cur].count++;
        }
        vector<ll> ans(tot+1);
        for(auto it=topo.rbegin();it!=topo.rend();it++){
            int i=(*it);
            for(auto x:nodes[i].has) ans[x]=nodes[i].count;
            nodes[nodes[i].fail].count+=nodes[i].count;
        }
        for(int i=1;i<=tot;++i) cout<<ans[i]<<"\n";
    }
};

void solve(){
    ll n;
    cin>>n;
    ACAutomation ac;
    fp(i,1,n){
        string s;
        cin>>s;
        ac.insert(s);
    }
    ac.build();
    string s;
    cin>>s;
    ac.query(s);
}
```

## 6.6 SA

```cpp
struct SA{
    string s;
    int n;
    vector<int> sa;
    vector<int> rk;
    vector<int> height;

    //传0-base的字符串
    SA(const string& str){
        n=str.size();
        s=" "+str;
        sa.resize(n+1);
        rk.resize(2*n+1);
        height.resize(n+1);
        build();
    }

    void build(){
        int m=255; //计数桶的个数
        vector<int> oldrk(2*n+1,0);
        vector<int> cnt(max(n,m)+1,0);
        vector<int> id(n+1);
        for(int i=1;i<=n;++i){
            rk[i]=s[i];
            cnt[rk[i]]++;
        }
        for(int i=1;i<=m;++i){
            cnt[i]+=cnt[i-1];
        }
        for(int i=n;i>=1;--i){
            sa[cnt[rk[i]]--]=i;
        }
        for(int k=1;k<n;k*=2){
            int num=0;
            for(int i=n-k+1;i<=n;++i) id[++num]=i;
            for(int i=1;i<=n;++i){
                if(sa[i]>k) id[++num]=sa[i]-k;
            }

            fill(cnt.begin(),cnt.begin()+m+1,0);
            for(int i=1;i<=n;++i){
                cnt[rk[id[i]]]++;
            }
            for(int i=1;i<=m;++i){
                cnt[i]+=cnt[i-1];
            }
            for(int i=n;i>=1;--i){
                sa[cnt[rk[id[i]]]--]=id[i];
            }
    
            swap(rk,oldrk);
            int p=0;
            for(int i=1;i<=n;++i){
                if(oldrk[sa[i]]==oldrk[sa[i-1]] and oldrk[sa[i] + k] == oldrk[sa[i - 1] + k]){
                    rk[sa[i]] = p;
                }
                else{
                    rk[sa[i]] = ++p;
                }
            }
    
            if(p==n) break;
            m=p;
        }

        if(n==1) rk[1]=1;

        int k=0;
        for(int i=1;i<=n;++i){
            if(rk[i]==1) {
                k=0;
                height[rk[i]]=0;
                continue;
            }
            if(k) k--;
            int j=sa[rk[i]-1];
            while(i+k<=n and j+k<=n and s[i+k]==s[j+k]) k++;
            height[rk[i]]=k;
        }
    }
};

void solve(){
    string s;
    cin>>s;
    SA sa(s);
    int n=s.size();
    fp(i,1,n) cout<<sa.sa[i]<<" ";
}
```



# 6. DP

## 6.1 子集枚举

单纯子集枚举为$O(3^{n})$的复杂度。

```cpp
void solve(){
    ll n,c;
    cin>>n>>c;
    vector<ll> w(n);
    fp(i,0,n-1) cin>>w[i];
    vector<bool> valid(1LL<<n);
    vector<ll> dp(1LL<<n,INF);
    dp[0]=0;
    fp(i,1,(1LL<<n)-1){
        ll cur=0;
        fp(j,0,n-1) if((1LL<<j)&i) cur+=w[j];
        if(cur<=c) valid[i]=1;
    }
    fp(mask,1,(1LL<<n)-1){
        for(int sub=mask;sub;sub = ((sub-1) & mask)){
            if(valid[sub]){
                dp[mask]=min(dp[mask],dp[mask^sub]+1);
            }
        }
    }
    cout<<dp[(1LL<<n)-1];
}
```

## 6.2 斜率优化

```cpp
void solve(){
    ll n,c;
    cin>>n>>c;
    vector<ll> h(n+1);
    fp(i,1,n) cin>>h[i];
    vector<ll> dp(n+1);
    vector<ll> x(n+1);
    vector<ll> y(n+1);
    // xi=hi yi=hi^2+dp[i]

    deque<pll> pnt; // point
    auto check = [&](const pll & u, const pll & v, const pll & w){  //如果要删，return 1
        pll k1 = {v.fi - u.fi, v.se - u.se};
        pll k2 = {w.fi - v.fi, w.se - v.se};
        //根据题意调整是否可以取等，不取等会保留相同斜率
        return ((lll)k1.se * k2.fi > (lll)k1.fi * k2.se);  
    };
    
    dp[1]=0;
    x[1]=h[1];
    y[1]=h[1]*h[1]+dp[1];
    pnt.push_back({x[1],y[1]});
    fp(i,2,n){
        while(pnt.size()>=2){
            const pll & u = pnt.front();
            const pll & v = pnt[1];
            ll k = 2 * h[i];
            pll k1 = {v.fi - u.fi, v.se - u.se};
            if(k1.se < (lll) k1.fi * k) pnt.pop_front(); 
            else break;
        }
        dp[i]=h[i]*h[i]+(pnt[0].se-2*h[i]*pnt[0].fi)+c;
        x[i]=h[i];
        y[i]=h[i]*h[i]+dp[i];
        while(pnt.size()>=2 and check(pnt[pnt.size()-2],pnt.back(),{x[i],y[i]})){
            pnt.pop_back();
        }
        pnt.push_back({x[i],y[i]});
    }
    cout<<dp[n]<<"\n";
}
```

# 7. 博弈论

## 7.1 Nim博弈

给定 $n$ 堆物品，第 $i$ 堆物品有 $a_i$ 个。两名玩家分别行动，每次可以任选一堆，取出任意多个物品，可以一把取光但是不能不取。取走最后一个物品的人胜利。

先手必胜，当且仅当$a_1\oplus a_2 \oplus … \oplus a_n =0$。

## 7.2 Wythoff博弈

有两堆石子，石子数可以不同。两人轮流取石子，每次可以在一堆中取，或者从两堆中取走相同个数的石子，数量不限，取走最后一个石头的人获胜。判定先手是否必胜。

假设两堆石子为$ (x,y),x<y$。

那么先手必败，当且仅当：

$(y-x)*\frac{\sqrt 5 +1}{2}=x$

下面是使用整数，避免浮点误差的做法。

```cpp
void solve(){
	ll x,y;
    cin>>x>>y;
    if(x>y) swap(x,y);
    ll k=y-x;
    if((2*x-k)*(2*x-k)<=5*k*k and 5*k*k<(2*x+2-k)*(2*x+2-k)){
        cout<<0<<"\n"; //先手必败
    }
    else{
        cout<<1<<"\n"; //先手必胜
    }
}
```

# 8. 杂项

## 8.1 最长上升子序列

```cpp
void solve(){
    ll n;
    cin>>n;
    vector<ll> a(n+1);
    for(int i=1;i<=n;++i) {cin>>a[i];}
    vector<ll> q;
    for(int i=1;i<=n;++i){
        // 严格上升用lower_bound，单调不减用upper_bound
        auto it=lower_bound(q.begin(),q.end(),a[i]);
        if(it==q.end()) q.push_back(a[i]);
        else {
            *it=a[i];
        }
    }
    cout<<q.size();
}
```

## 8.2 三分

### 8.2.1 三分（三等分）

```cpp
vector<double> coe(N);
ll n;

double f(double x){
    double res=0;
    for(int i=n;i>=0;--i){
        res+=coe[i]*pow(x,i);
    }
    return res;
}

void solve(){
    cin>>n;
    double l,r;
    cin>>l>>r;
    double l1,r1;
    l1=l+(r-l)/3;
    r1=r-(r-l)/3;
    for(int i=n;i>=0;--i){
        cin>>coe[i];
    }
    while(r1-l1>eps){
        l1=l+(r-l)/3;
        r1=r-(r-l)/3;
        double fl1=f(l1);
        double fr1=f(r1);
        if(fl1>=fr1){
            r=r1;
        }
        else{
            l=l1;
        }
    }
    cout<<l1;
}
```

### 8.2.2 三分（近似三分）

```cpp
vector<double> coe(N);
ll n;

double f(double x){
    double res=0;
    for(int i=n;i>=0;--i){
        res+=coe[i]*pow(x,i);
    }
    return res;
}

void solve(){
    cin>>n;
    double l,r;
    cin>>l>>r;
    double l1,r1;
    l1=(l+r)/2-eps;
    r1=(l+r)/2+eps;
    for(int i=n;i>=0;--i){
        cin>>coe[i];
    }
    ll cnt=0;
    while(cnt<=100){
        cnt++;
        l1=(l+r)/2-eps;
        r1=(l+r)/2+eps;
        double fl1=f(l1);
        double fr1=f(r1);
        if(fl1>=fr1){
            r=r1;
        }
        else{
            l=l1;
        }
    }
    cout<<l1;
}
```

### 8.2.3 三分（整数三分）

```cpp
void solve(){
    ll n,m;
    cin>>n>>m;
    vector<ll> a(n+1);
    vector<ll> b(m+1);
    for(int i=1;i<=n;++i){
        cin>>a[i];
    }
    for(int i=1;i<=m;++i){
        cin>>b[i];
    }
    sort(a.begin()+1,a.end());
    sort(b.begin()+1,b.end());
    vector<ll> asum(n+1);
    vector<ll> bsum(m+1);
    for(int i=1;i<=n;++i){
        asum[i]=asum[i-1]+a[n+1-i]-a[i];
    }
    for(int i=1;i<=m;++i){
        bsum[i]=bsum[i-1]+b[m+1-i]-b[i];
    }
    vector<ll> ans;
    for(ll k=1;max(0LL,2*k-n)<=min(k,m-k);++k){
        ll l=max(0LL,2*k-n);
        ll r=min(k,m-k);
        auto f=[&](ll u){return bsum[u]+asum[k-u];};
        while(r-l>3){
            ll ml=(2*l+r)/3;
            ll mr=(l+2*r)/3;
            if(f(ml)<f(mr)){
                l=ml;
            }
            else{
                r=mr;
            }
        }
        ll res=0;
        for(int i=l;i<=r;++i){
            res=max(res,f(i));
        }
        ans.push_back(res);
    }
    cout<<ans.size()<<"\n";
    for(auto &p:ans) cout<<p<<" ";
    cout<<"\n";
}
```

## 8.3 CDQ分治

```cpp
struct it{
    ll a,b,c,cnt,ans;
};

void solve(){
    ll n,k;
    cin>>n>>k;
    vector<array<ll,3>> a(n+1);
    fp(i,1,n){
        cin>>a[i][0]>>a[i][1]>>a[i][2];
    }
    sort(a.begin()+1,a.end());
    ll m=0;
    vector<it> its;
    its.push_back({0,0,0,0,0});
    fp(i,1,n){
        if(a[i][0]==a[i-1][0] and a[i][1]==a[i-1][1] and a[i][2]==a[i-1][2]){
            its[m].cnt++;
        }
        else{
            m++;
            its.push_back({a[i][0],a[i][1],a[i][2],1,0});
        }
    }
    vector<it> tmp(m+1);
    Fenwick<ll> bit(k);
    auto solve=[&](auto&& self,ll l,ll r) ->void {
        if(l==r) return;
        ll mid=l+r>>1;
        self(self,l,mid);
        self(self,mid+1,r);
        ll pnt1=l;
        ll pnt2=mid+1;
        ll pnt3=l;
        vector<pll> op;
        while(pnt1<=mid and pnt2<=r){
            if(its[pnt1].b<=its[pnt2].b){
                bit.add(its[pnt1].c,its[pnt1].cnt);
                op.push_back({its[pnt1].c,its[pnt1].cnt});
                tmp[pnt3]=its[pnt1];
                pnt1++;
                pnt3++;
            }
            else{
                its[pnt2].ans+=bit.rangeSum(1,its[pnt2].c);
                tmp[pnt3]=its[pnt2];
                pnt2++;
                pnt3++;
            }
        }
        while(pnt1<=mid){
            tmp[pnt3]=its[pnt1];
            pnt1++;
            pnt3++;
        }
        while(pnt2<=r){
            its[pnt2].ans+=bit.rangeSum(1,its[pnt2].c);
            tmp[pnt3]=its[pnt2];
            pnt2++;
            pnt3++;
        }
        for(auto [pos,cnt]:op) bit.add(pos,-cnt);
        fp(i,l,r) its[i]=tmp[i];
    };
    solve(solve,1,m);
    fp(i,1,m){
        its[i].ans+=its[i].cnt-1;
    }
    vector<ll> ans(n);
    fp(i,1,m) ans[its[i].ans]+=its[i].cnt;
    fp(i,0,n-1) cout<<ans[i]<<"\n";
}
```

## 8.4 离散化

```cpp
sort(vec.begin(), vec.end());
vec.erase(unique(vec.begin(), vec.end()), vec.end());
```

## 8.5 随机数

```cpp
mt19937_64 rng(chrono::steady_clock::now().time_since_epoch().count());

ll rand(ll l, ll r) {
    uniform_int_distribution<ll> distribution(l, r);
    return distribution(rng);
}
```

## 8.6 lll的输出

```cpp
void solve(){
    ll x;
    cin>>x;
    lll x1=(lll) x*x;
    cout << format("{}", x1) << "\n";
}
```

## 8.7 对拍

```cpp
int main(){
	ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
	ll cnt=0;
	while(1){
		system("generator.exe > in.txt");
		system("std.exe < in.txt > std.txt");
		system("my.exe < in.txt > my.txt");
		if(system("fc std.txt my.txt")) break;
		cout<<"AC"<<++cnt<<endl;
	}
	cout<<"WA"<<endl;
	return 0;
} 
```

bat版

```bat
@echo off
echo Compiling programs...
g++ generator.cpp -o generator.exe -O2 -std=c++23
g++ std.cpp -o std.exe -O2 -std=c++23
g++ my.cpp -o my.exe -O2 -std=c++23

if errorlevel 1 (
    echo Compilation failed!
    pause
    exit
)

echo Compilation successful!

:: 对拍循环
set /a count=0
:loop
set /a count+=1
echo Testing case %count%...

:: 生成数据
generator.exe > data.in

:: 运行两个程序
std.exe < data.in > std.out
my.exe < data.in > my.out

:: 比较输出
fc std.out my.out > nul

if errorlevel 1 (
    echo ========================================
    echo Found different output at case %count%!
    echo ========================================
    echo Input data saved in: data.in
    echo Standard output saved in: std.out
    echo Your output saved in: my.out
    pause
    exit
) else (
    echo Case %count% passed.
)

goto loop
```

## 8.8 设置输出精度

```cpp
cout << fixed << setprecision(15);
```

## 8.9 pbds的平衡树

```cpp
tree<int, null_type, less<int>,rb_tree_tag,tree_order_statistics_node_update> tr;
```

