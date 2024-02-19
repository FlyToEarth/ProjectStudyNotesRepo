lowbit(x):返回x的最后一位1

x = 1010 lowbit(x) = 10

x = 101000 lowbit(x) = 1000

都是二进制数

x & -x

-x:正数x 取反 + 1

x & -x = x & (~x + 1)

x = 1010...(1)00...000

~x = 0101...(0)11...111

~x + 1 = 0101..(1)00...000

x & (~x + 1) = 0000..(1)00...000

应用：统计x里1的个数

```C++
#include <iostream>

using namespace std;

int lowbit(int x) {
    return x & (-x);
}

int main() {
    int n;
    cin >> n;
    while (n--) {
        int x;
        cin >> x;
        int res = 0;
        while (x) {
            x -= lowbit(x);
            res++;
        }
    }
    cout << res << endl;
    return 0;
}
```

原码

x = 000...000 1010

反码

x = 1111...111 0101

补码

x = 反码 + 1

