# Report of Fifth Homework of Computer Network Course
<center>卓越二班-2016302580264-黎冠延</center>

## P1
### Codes:
```c++
#include <iostream>
#include <functional>
#include <unordered_set>
#include <vector>
using std::unordered_set;
using std::vector;
using std::cout;

class Point {
    char _name;
    unordered_set<Point *> _neighbours;
    
public:
    Point(char name): _name(name) { }
    
    char name() { return _name; }
    const unordered_set<Point *> &neighbours() { return _neighbours; }
    
    void add_neighbour(Point *p) {
        if (!p) return;
        _neighbours.emplace(p);
        p->_neighbours.emplace(this);
    }
    void add_neighbours(const vector<Point *> ps) {
        for (const auto &p: ps)
            add_neighbour(p);
    }
};

int find_paths(Point *ori, Point *tar, std::ostream &os = cout) {
    os << ori->name() << " -> " << tar->name() << ":" << std::endl;
    int count = 0;
    vector<Point *> path;
    unordered_set<Point *> pset;
    auto print_path = [&path, tar, &os, &count]() {
        for (auto &p: path)
            os << p->name() << " -> ";
        os << tar->name() << std::endl;
        ++count;
    };
    std::function<void(Point *)> inner_find = [tar, &path, &pset, &print_path, &inner_find](Point *o) {
        if (o == tar)
            print_path();
        else {
            path.push_back(o);
            pset.insert(o);
            for (auto &p: o->neighbours())
                if (pset.find(p) == pset.end())
                    inner_find(p);
            pset.erase(o);
            path.pop_back();
        }
    };
    inner_find(ori);
    return count;
}

void run() {
    Point u('u'), v('v'), w('w'), x('x'), y('y'), z('z');
    
    u.add_neighbours({&v, &w, &x});
    v.add_neighbours({&x, &w});
    w.add_neighbours({&x, &y, &z});
    x.add_neighbour(&y);
    y.add_neighbour(&z);
    
    find_paths(&y, &u);
    find_paths(&x, &z);
    find_paths(&z, &u);
    find_paths(&z, &w);
}

int main() {
    run();
    return 0;
}
```
### Answer:
y -> u:
y -> z -> w -> x -> v -> u, 
y -> z -> w -> x -> u, 
y -> z -> w -> v -> x -> u, 
y -> z -> w -> v -> u, 
y -> z -> w -> u, 
y -> x -> w -> v -> u, 
y -> x -> w -> u, 
y -> x -> v -> w -> u, 
y -> x -> v -> u, 
y -> x -> u, 
y -> w -> x -> v -> u, 
y -> w -> x -> u, 
y -> w -> v -> x -> u, 
y -> w -> v -> u, 
y -> w -> u, 

## P2
According to codes in P1:

x -> z:
x -> y -> z, 
x -> y -> w -> z, 
x -> w -> z, 
x -> w -> y -> z, 
x -> v -> w -> z, 
x -> v -> w -> y -> z, 
x -> v -> u -> w -> z, 
x -> v -> u -> w -> y -> z, 
x -> u -> w -> z, 
x -> u -> w -> y -> z, 
x -> u -> v -> w -> z, 
x -> u -> v -> w -> y -> z, 

z -> u:
z -> y -> x -> w -> v -> u, 
z -> y -> x -> w -> u, 
z -> y -> x -> v -> w -> u, 
z -> y -> x -> v -> u, 
z -> y -> x -> u, 
z -> y -> w -> x -> v -> u, 
z -> y -> w -> x -> u, 
z -> y -> w -> v -> x -> u, 
z -> y -> w -> v -> u, 
z -> y -> w -> u, 
z -> w -> y -> x -> v -> u, 
z -> w -> y -> x -> u, 
z -> w -> x -> v -> u, 
z -> w -> x -> u, 
z -> w -> v -> x -> u, 
z -> w -> v -> u, 
z -> w -> u, 

z -> w:
z -> y -> x -> w, 
z -> y -> x -> v -> w, 
z -> y -> x -> v -> u -> w, 
z -> y -> x -> u -> w, 
z -> y -> x -> u -> v -> w, 
z -> y -> w, 
z -> w, 

## P3
### Codes
```c++
#include <iostream>
#include <vector>
#include <memory>
#include <unordered_map>
#include <unordered_set>
#include <functional>
#include <optional>
#include <iomanip>
#include <string>
using std::cout;
using std::string;
using std::function;
using std::unordered_set;
using std::unordered_map;
using std::shared_ptr;
using std::vector;
using std::pair;

class Point {
    char _name;
    unordered_map<Point *, size_t> _neighbours;
    
public:
    Point(char name): _name(name) { }
    
    char name() const { return _name; }
    const unordered_map<Point *, size_t> &neighbours() const { return _neighbours; }
    
    void add_neighbour(Point *p, size_t distance = -1) {
        if (!p) return;
        _neighbours.emplace(p, distance);
        p->_neighbours.emplace(this, distance);
    }
    void add_neighbours(const vector<Point *> &ps) {
        for (const auto &p: ps)
            add_neighbour(p);
    }
    void add_neighbours(const vector<Point *> &ps, const vector<size_t> &distance) {
        if (ps.size() != distance.size()) throw 1;
        for (size_t i = 0; i < ps.size(); ++i)
            add_neighbour(ps[i], distance[i]);
    }
};

void simple_Dijkstra_process_output(Point *f, std::ostream &os = cout) {
    // get_point_list
    unordered_map<Point *, std::optional<pair<size_t, Point *>>> points;
    string path;
    size_t count = 0;
    size_t offset = 0;
    auto print_contemporary = [&points, &count, &os, &path]() -> void {
        os << "|" << count++ << "|";
        os << path << "|";
        for (auto &p: points)
            if (!p.second)
                os << "$\\infty$|";
            else if (p.second->first)
                os << p.second->first << ", " << p.second->second->name() << "|";
            else
                os << " |";
        os << std::endl;
    };
    auto get_min = [&points, &path]() -> pair<size_t, Point *> {
        assert(path.size() < points.size() + 1);
        size_t i = -1;
        Point *p = nullptr;
        for (auto iit = points.begin(); iit != points.end(); ++iit)
            if (iit->second && iit->second->first)
                if (p == nullptr || iit->second->first < i) {
                    i = iit->second->first;
                    p = iit->first;
                }
        return std::make_pair(i, p);
    };
    auto update = [&points, &path, &offset, f](Point *p) -> void {
        path += p->name();
        if (p != f)
            points[p]->first = 0;
        for (const auto &pp: p->neighbours()) {
            if (pp.first == f) continue;
            auto &k = points[pp.first];
            if (!k || k->first > pp.second + offset)
                k = std::make_pair(pp.second + offset, (Point *)p);
        }
    };
    {
        std::function<void (const Point *)> dfs = [&points, &dfs](const Point *p) {
            points.emplace((Point *)p, std::nullopt);
            for (const auto &pp: p->neighbours())
                if (points.find(pp.first) == points.end())
                    dfs(pp.first);
        };
        dfs(f);
        points.erase((Point *)(f));
    }
    // output header
    {
        os << "|step|N'|";
        for (auto &p: points)
            os << "D(" << p.first->name() << "), p(" << p.first->name() << ")|";
        os << std::endl;
        os << "|--|--|";
        for (auto &p: points)
            os << "--|";
        os << std::endl;
    }
    // initialize
    while (true) {
        update(f);
        print_contemporary();
        if (path.size() >= points.size() + 1) break;
        auto info = get_min();
        offset += info.first;
        f = info.second;
    }
}

void run() {
    Point t('t'), u('u'), v('v'), w('w'), x('x'), y('y'), z('z');
    
    t.add_neighbours({&u, &v, &y}, {2, 4, 7});
    u.add_neighbours({&v, &w}, {3, 3});
    v.add_neighbours({&w, &x, &y}, {4, 3, 8});
    w.add_neighbour(&x, 6);
    x.add_neighbours({&y, &z}, {6, 8});
    y.add_neighbour(&z, 12);

    // P3
    simple_Dijkstra_process_output(&x);
    
    // P4
    simple_Dijkstra_process_output(&t);
    simple_Dijkstra_process_output(&u);
    simple_Dijkstra_process_output(&v);
    simple_Dijkstra_process_output(&w);
    simple_Dijkstra_process_output(&y);
    simple_Dijkstra_process_output(&z);
}
```
### Answer
|step|N'|D(t), p(t)|D(u), p(u)|D(y), p(y)|D(w), p(w)|D(v), p(v)|D(z), p(z)|
|--|--|--|--|--|--|--|--|
|0|x|$\infty$|$\infty$|6, x|6, x|3, x|8, x|
|1|xv|7, v|6, v|6, x|6, x| |8, x|
|2|xvu|7, v| |6, x|6, x| |8, x|
|3|xvuy|7, v| | |6, x| |8, x|
|4|xvuyw|7, v| | | | |8, x|
|5|xvuywt| | | | | |8, x|
|6|xvuywtz| | | | | | |

## P4
According to codes in P3.
### a.
|step|N'|D(u), p(u)|D(z), p(z)|D(y), p(y)|D(w), p(w)|D(v), p(v)|D(x), p(x)|
|--|--|--|--|--|--|--|--|
|0|t|2, t|$\infty$|7, t|$\infty$|4, t|$\infty$|
|1|tu| |$\infty$|7, t|5, u|4, t|$\infty$|
|2|tuv| |$\infty$|7, t|5, u| |9, v|
|3|tuvw| |$\infty$|7, t| | |9, v|
|4|tuvwy| |30, y| | | |9, v|
|5|tuvwyx| |30, y| | | | |
|6|tuvwyxz| | | | | | |
### b.
|step|N'|D(t), p(t)|D(v), p(v)|D(z), p(z)|D(x), p(x)|D(w), p(w)|D(y), p(y)|
|--|--|--|--|--|--|--|--|
|0|u|2, u|3, u|$\infty$|$\infty$|3, u|$\infty$|
|1|ut| |3, u|$\infty$|$\infty$|3, u|9, t|
|2|utv| | |$\infty$|8, v|3, u|9, t|
|3|utvw| | |$\infty$|8, v| |9, t|
|4|utvwx| | |24, x| | |9, t|
|5|utvwxy| | |24, x| | | |
|6|utvwxyz| | | | | | |
### c.
|step|N'|D(u), p(u)|D(t), p(t)|D(w), p(w)|D(y), p(y)|D(x), p(x)|D(z), p(z)|
|--|--|--|--|--|--|--|--|
|0|v|3, v|4, v|4, v|8, v|3, v|$\infty$|
|1|vu| |4, v|4, v|8, v|3, v|$\infty$|
|2|vux| |4, v|4, v|8, v| |14, x|
|3|vuxt| | |4, v|8, v| |14, x|
|4|vuxtw| | | |8, v| |14, x|
|5|vuxtwy| | | | | |14, x|
|6|vuxtwyz| | | | | | |
### d.
|step|N'|D(t), p(t)|D(u), p(u)|D(v), p(v)|D(z), p(z)|D(y), p(y)|D(x), p(x)|
|--|--|--|--|--|--|--|--|
|0|w|$\infty$|3, w|4, w|$\infty$|$\infty$|6, w|
|1|wu|5, u| |4, w|$\infty$|$\infty$|6, w|
|2|wuv|5, u| | |$\infty$|15, v|6, w|
|3|wuvt| | | |$\infty$|15, v|6, w|
|4|wuvtx| | | |26, x|15, v| |
|5|wuvtxy| | | |26, x| | |
|6|wuvtxyz| | | | | | |
### e.
|step|N'|D(t), p(t)|D(u), p(u)|D(w), p(w)|D(x), p(x)|D(v), p(v)|D(z), p(z)|
|--|--|--|--|--|--|--|--|
|0|y|7, y|$\infty$|$\infty$|6, y|8, y|12, y|
|1|yx|7, y|$\infty$|12, x| |8, y|12, y|
|2|yxt| |15, t|12, x| |8, y|12, y|
|3|yxtv| |15, t|12, x| | |12, y|
|4|yxtvw| |15, t| | | |12, y|
|5|yxtvwz| |15, t| | | | |
|6|yxtvwzu| | | | | | |
### f.
|step|N'|D(t), p(t)|D(u), p(u)|D(x), p(x)|D(w), p(w)|D(y), p(y)|D(v), p(v)|
|--|--|--|--|--|--|--|--|
|0|z|$\infty$|$\infty$|8, z|$\infty$|12, z|$\infty$|
|1|zx|$\infty$|$\infty$| |14, x|12, z|11, x|
|2|zxv|23, v|22, v| |14, x|12, z| |
|3|zxvy|23, v|22, v| |14, x| | |
|4|zxvyw|23, v|22, v| | | | |
|5|zxvywu|23, v| | | | | |
|6|zxvywut| | | | | | |

## P5
### Codes
```c++
#include <iostream>
#include <unordered_map>
#include <unordered_set>
#include <functional>
using std::cout;
using std::unordered_map;
using std::unordered_set;

class Point {
    char _name;
    unordered_map<Point *, size_t> _neighbours;
    
public:
    Point(char name): _name(name) { }
    
    char name() const { return _name; }
    const unordered_map<Point *, size_t> &neighbours() const { return _neighbours; }
    
    void add_neighbour(Point *p, size_t distance = -1) {
        if (!p) return;
        _neighbours.emplace(p, distance);
        p->_neighbours.emplace(this, distance);
    }
    void add_neighbours(const vector<Point *> &ps) {
        for (const auto &p: ps)
            add_neighbour(p);
    }
    void add_neighbours(const vector<Point *> &ps, const vector<size_t> &distance) {
        if (ps.size() != distance.size()) throw 1;
        for (size_t i = 0; i < ps.size(); ++i)
            add_neighbour(ps[i], distance[i]);
    }
};

template <typename ActTp, typename OverPredicateTp>
void dfs(Point *s, ActTp &&act, OverPredicateTp &&op) {
    unordered_set<Point *> pset;
    std::function<void (Point *)> idfs = [&pset, &act, &op, &idfs](Point *p) -> void {
        if (op(p)) return;
        pset.emplace(p);
        act(p);
        for (auto &pp: p->neighbours())
            if (pset.find(pp.first) == pset.end())
                idfs(pp.first);
    };
    idfs(s);
}

unordered_map<Point *, size_t> cal_distanc_vector(Point *f) {
    unordered_map<Point *, size_t> r;
    auto update = [&r](Point *p) -> void {
        size_t offset = r[p];
        for (auto &pp: p->neighbours()) {
            auto it = r.find(pp.first);
            if (it == r.end()) r.emplace(pp.first, pp.second + offset);
            else if(it->second > offset + pp.second)
                it->second = pp.second + offset;
        }
    };
    r[f] = 0;
    dfs(f, update, [](Point *){ return false; });
    return r;
}

void run() {
    Point t('t'), u('u'), v('v'), w('w'), x('x'), y('y'), z('z');
    
    u.add_neighbours({&v, &y}, {1, 2});
    v.add_neighbours({&x, &z}, {3, 6});
    x.add_neighbours({&y, &z}, {3, 2});
    
    auto &&r = cal_distanc_vector(&z);
    cout << "|";
    for (auto &p: r)
        cout << p.first->name() << "|";
    cout << std::endl;
    cout << "|";
    for (auto &p: r)
        cout << "--|";
    cout << std::endl;
    cout << "|";
    for (auto &p: r)
        cout << p.second << "|";
    cout << std::endl; 
}

int main() {
    run();
    return 0;
}
```
### Answer
|u|y|v|x|z|
|--|--|--|--|--|
|6|5|5|2|0|
