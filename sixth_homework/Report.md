# Report of Sixth Homework of Computer Network Course
<center>卓越二班-2016302580264-黎冠延</center>

## P1
1110 1<br>
0110 0<br>
1001 0<br>
1101 1<br>

1100 0

## P2
Let the data be:

0101 0<br>
1101 1<br>

1000 1

### Correction
Let the error data be:

0111 0
1101 1

1000 1

So, [0, 2] is errorous, because line 0 and column 2 has error.
And should be 0.

### Detection
Let the error data be:

0111 0<br>
1111 1<br>

1000 1

So, both line 0 and line 1 have error(s), but no column has error.
So where is wrong can't be distinguished.

## P3
### Codes
```C++
#include <iostream>
#include <string>
using std::string;
using std::cout;
using std::endl;


template <typename t>
void print_binary(t &&o) {
    char *c = (char *)&o;
    for (size_t i = sizeof(o) - 1; i < sizeof(o); --i) {
        for (char j = 7; j >= 0; --j)
            cout << !!(c[i] & char(1 << j));
        cout << " ";
    }
    cout << endl;
}

template <typename FArg, typename ...Args>
void print_binary(FArg &&fa, Args&&... args) {
    print_binary(fa);
    cout << std::endl;
    print_binary(args...);
}

template <typename NumTp = unsigned short>
NumTp cal_Checksum(const string &s) {
    const static bool IS_BIG_ENDIAN =
        static_cast<unsigned short>(0x1234) != *(unsigned short *)"\x12\x34";
    static_assert(!std::is_signed_v<NumTp>, "This cannot be signed");
    NumTp r = 0;
    for (size_t i = 0; i < s.size(); ++i) {
        NumTp tmp = s[i];
        if (IS_BIG_ENDIAN)
            for (size_t j = 1; j < sizeof(NumTp); ++j) {
                tmp <<= 8;
                if (i + 1 < s.size()) tmp |= s[++i];
            }
        r += tmp + (tmp > NumTp(~0) - r);
    }
    return r == NumTp(~0) ? r : r ^ NumTp(~0);
}

template <typename ...Args>
void calAndPrint(Args&&... args) {
    (print_binary(cal_Checksum(args)), ...);
}

void run() {
    calAndPrint("Networking");
}

int main() {
    run();
}

```

### Answer
11110011 11011111

## P4
### Codes
The same code as before with a slight difference in run():
```C++
void run() {
    string a;
    string b;
    string c;
    for (char c = 1; c <= 10; ++c)
        a += c;
    for (char cc = 'B'; cc <= 'K'; ++cc) {
        b += cc;
        c += cc - 'A' + 'a';
    }
    
    calAndPrint(a, b, c);
}
```

### Answer
#### a.
11100110 11100001 

#### b.
10100000 10011011 

#### c.
11111111 11111010


## P5
### Codes
```C++
#include <iostream>
using std::cout;
using std::endl;

template <typename t>
void print_binary(t &&o) {
    char *c = (char *)&o;
    for (size_t i = sizeof(o) - 1; i < sizeof(o); --i) {
        for (char j = 7; j >= 0; --j)
            cout << !!(c[i] & char(1 << j));
        cout << " ";
    }
    cout << endl;
}

template <typename NumTp = unsigned int>
std::pair<NumTp, NumTp> binary_divide(NumTp a, NumTp b) {
    static_assert(std::is_unsigned_v<NumTp> && !std::is_floating_point_v<NumTp>, "This must be unsigned and not floating point");
    assert(b);
    std::pair<NumTp, NumTp> r(0, a);
    size_t shift_pos = sizeof(NumTp) * 8;
    for (NumTp tmp = b; tmp; tmp >>= 1) --shift_pos;
    while (r.second >= b) {
        while (b << shift_pos > r.second && shift_pos < sizeof(NumTp) * 8) --shift_pos;
        r.first |= 1 << shift_pos;
        r.second ^= b << shift_pos;
        --shift_pos;
    }
    return r;
}

template <typename NumTp>
NumTp get_CRC(NumTp msg, NumTp G) {
    static_assert(std::is_unsigned_v<NumTp> && !std::is_floating_point_v<NumTp>, "This must be unsigned and not floating point");
    assert(G);
    for (NumTp n = G >> 1; n; n >>= 1)
        if (msg & (1 << (sizeof(NumTp) * 8 - 1))) assert(false);
        else msg <<= 1;
    return binary_divide(msg, G).second;
}

void run() {
    print_binary(get_CRC<unsigned short>(0b1010101010, 0b10011));
}
```

#### Answer
00000000 00000100

That is:

0100
