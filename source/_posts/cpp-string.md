---
title: cpp string
date: 2024-03-11 10:34:27
tags:
---

# 优雅切割
* 默认空格
```cpp
#include <string>
#include <sstream>
#include <iostream>

using namespace std;

int main() {
    istringstream iss("capiTalIze   tHe titLe");
    string s;
    while (iss >> s) {
        std::cout << "size = "<< s.size() << " " << s << std::endl;
    }
    return 0;
}
```
* 指定字符
```cpp
#include <string>
#include <sstream>
#include <iostream>

using namespace std;

int main() {
    istringstream iss("capiTalIze,    tHe, titLe");
    string s;
    while (getline(iss, s, ',')) {
        std::cout << "size = "<< s.size() << " " << s << std::endl;
    }
    return 0;
}
```
* 优雅format
```cpp
#include <string>
#include <sstream>
#include <iostream>

using namespace std;

int main() {
    istringstream iss("1 2 3");
    int i, j, k;
    iss >> i >> j >> k;
    cout << "i=" << i << ", j=" << j << ", k=" << k << endl;
    return 0;
}
```
