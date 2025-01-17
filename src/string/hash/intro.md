# Introduction to Rolling Hash

## 前言

讀完這段 Rolling Hash 的簡介後，你就足以處理多數的問題了！但如果你求知若渴，不妨把後續的章節也讀完 XDD

## How to calculate Rolling Hash

Hash (哈希)，是一種將資料壓縮成一個數字的方法，好比一個人 (完整的資料) 跟他的指紋 (哈希)，藉此加速字串匹配。而在競技程式設計中，最常用的方法是 Polynomial Hash (多項式哈希)，在台灣更多被稱作 Rolling Hash (滾動哈希)。

### 如何計算

多項式哈希（Polynomial Hash）是一種常用的哈希算法，它通常用於字串匹配、文本搜索、比較和壓縮等應用場景。其基本思想是將字串看作一個多項式，並使用多項式求值的方式來計算字串的哈希值。

以下是計算 Polynomial hash 的基本步驟：

首先，需要選擇一個適合的模數值 \\(m\\)，用於取模運算。

然後，需要選擇一個固定的基數 \\(a\\)，用於計算多項式中各項的次方。

通常 \\( (m, a) \\) 會是兩個互質的大數字，後續會分享如何選擇洽當的 \\( (m, a) \\)。

將字串中的每個字符轉換為一個對應的數字（通常使用 ASCII 碼值，或是對應成 0-25），作為多項式中相應項的係數。

一邊計算 \\(a\\) 的次方，一邊將多項式中各項的係數和次方相乘，同時將所有項的結果相加，最後對模數值 m 取模，即可得到最終的哈希值。

$$ a^{0} \cdot S_0 + a^{1} \cdot S_1 + a^{2} \cdot S_2 ... \mod m = H $$

\\(S\\) 是字串，而 \\(S_0\\) 是 `S[0]`，\\(H\\) 是計算出來的哈希值。

### 範例

例如，對於一個字串 s = "hello"，假設基數 a = 31，模數值 m = 10^9 + 7，則可以按照以下方式計算其 Polynomial hash：

將字串 s 轉換為對應的數字序列，得到 [104, 101, 108, 108, 111]。

計算多項式中各項的次方，得到 [1, 31, 961, 29791, 923521]。

將各項的係數和次方相乘，得到 [104, 3131, 103788, 3217428, 102509832]。

將所有項的結果相加，得到 105834283。

對模數值 m 取模，得到最終的哈希值 105834283。

### Code Implementation

```C++
long long polynomial_hash(long long s, long long a, long long m) {
    long long h = 0, p = 1;
    for(int i = 0; i < S.size(); i++) {
        h = (h + (S[i] - 'a') * p) % m;
        p = (p * a) % m;
    }
    return h
}
```

### 判斷字串是否相等

如果兩片指紋相同，那代表（有很高的機率）這兩片指紋來自同個人；兩個哈希值相同，那就（有很高的機率）代表這兩組資料相同。

而判斷字串是否相等，只需比對兩個哈希值 (Hash Value) 是否相等即可，舉例而言，有兩個字串 \\(S_1\\) 與 \\(S_2\\) 要判斷是否相等，那麼，判斷 `polynomial_hash(s_1, a, m) == polynomial_hash(s_2, a, m)` 即可。

## Hash Collision and Mathematical Analysis

哈希碰撞（Hash Collision）指的是兩個不同的輸入值在經過哈希函數計算後產生了相同的哈希值。由於哈希函數的值域通常比輸入值的集合小得多，因此哈希碰撞是不可避免的現象。

而在 Rolling Hash 中，哈希碰撞就會導致本不該相等的兩個字串，被視為相同的字串，最終使得程式的邏輯出現錯誤。

以 \\(a=1, m=2\\) 為例，字串 `ab` 與 `ba` 有相同的哈希值，進而兩者被視為相等的字串，但實際上這兩者並不相等，最終使得程式結果錯誤。

而要估計 Rolling Hash 的 Collision Rate，可以使用生日悖論（Birthday Paradox）的概念。

生日悖論指的是當有一群人在一起時，假設他們的生日是獨立均勻分布在一年中的每一天，那麼當這個群體的大小達到一定程度時，至少有兩個人的生日是相同的機率遠高於我們通常感覺到的機率。具體來說，當群體的大小為23人時，至少有兩人生日相同的概率已經超過 `50%`。

將生日悖論應用到哈希碰撞的情境中，假設我們有一個哈希函數 \\(h\\)，它將 \\(n\\) 個不同的輸入值映射到 \\(m\\) 個哈希值中的某一個。如果我們隨機選擇 \\(k\\) 個輸入值，那麼它們中至少有兩個輸入值的哈希值相同的概率就可以用生日悖論的公式計算：

$$ P(k,m)=1−\frac{(m−k)!m}{k \cdot m!} $$
​
其中 \\(P(k, m)\\) 表示選擇 \\(k\\) 個輸入值中至少有兩個輸入值的哈希值相同的概率，\\(m\\) 表示哈希值的空間大小。

對於 Polynomial Hash，哈希值空間的大小可以看作是模 \\(p\\) 的值，其中 \\(p\\) 是一個大質數。因此，我們可以使用上述公式來估計 Polynomial Hash 的 Collision Rate。

## Picking the correct modulo and base

首先，選定的 \\(m\\) 跟 \\(a\\) 必須要互質，假設 \\(m\\) 跟 \\(a\\) 不互質的話，就會造成 \\(a^1, a^2, ..., a^m\\) 中出現重複的數字，進而使得哈希碰撞的概率上升。

以 \\(m=8, a=6\\) 為例，\\(a^1=6, a^2=4, a^3=a^4=a^5=a^6=0\\)，可以觀察到有多個數字重複，因此必須選擇互質的 \\(m, a\\)。

其次，不要選擇 \\(2\\) 的冪次作為 \\(m\\)，這會讓出題者能夠構造一個 Thue-Morse Sequence 去攻擊你的 Hash，後續章節會介紹更多怎麼攻擊 Rolling Hash。

最後，建議選擇幾組冷門的 \\(m, a\\) 作為你的模板，許多出題者會對常用的 \\(m, a\\) 構造容易碰撞的測資，如果你選的 \\(m, a\\) 夠冷門（像是你的手機號碼），就不容易被出題者猜中。下面是幾個常見的 \\(m, a\\)，請盡量不要選擇這些 \\(m, a\\)。

1. \\(m = 10^N+7\\)
2. \\(m = 2^N+1\\)
3. \\(m = 2 \cdot 3 \cdot 5 \cdot 7 ...\\)，然後 \\(a\\) 選再下一個質數
4. \\(a = 2, 3, 4, 5, 6, 7, 8, 9, 10\\)

## Get the hash value of a substring

如何計算字串 \\(S\\) substring 的 hash value 呢？最直觀的，直接計算 `polynomial_hash(S[i:j], a, m)` 即可，但這樣做並不效率。

此時，使用 prefix sum 能夠減少計算 substring 的額外開銷，具體作法如下。

1. 建立一個數列 \\(H[i] = polynomial_hash(S[0:i], a, m)\\)
2. 初始設定 \\(H[0] = 0\\)，\\(H[i] = H[i - 1] + S[i] \cdot a^i \mod m\\)，並遞推這個數列
3. 計算 \\((H[j] - H[i]) \cdot a^{-i}\\) 即為 `polynomial_hash(S[i:j], a, m)`，計算 \\(a^{-i}\\) 時，可以參考模逆元的作法。

```C++
vector<long long> preprocess_substring(string S) {
    vector<long long> H = {0};
    long long p = 1;
    for (int i = 0; i < S.size(); i++) {
        H.push_back(((H[i] + S[i] - 'A') * p) % m);
        p = (p * a) % m;
    }
    return H;
}


long long mod_inverse(long long x, long long mod); // Returns the modulo inverse of x under mod
long long pow(long long x, long long p, long long mod); // Returns x^{p} under mod

long long substring_hash(vector<long long> H, int l, int r) {
    difference = (H[r] - H[l] + m) % m;
    return (difference * mod_inverse(pow(a, i, m), m)) % m
}
```

這樣做的話，只要先付出 \\(O(N)\\) 的時間建表，每次查詢 substring 就只要 \\(O(1)\\) 了。

## Compare the string lexicographically (by binary search)

給定兩個相同長度的字串 \\(X\\) 與 \\(Y\\)，判斷兩者的字典序的方法如下

1. 找到最小的 \\(i \geq 0\\)，使得 \\(X[i] \neq Y[i]\\)，如果找不到的話，這兩個字串必然相等
2. 判斷是否 \\(X[i] > Y[i]\\)

平常比大小時，找最小的 \\(i\\) 就很樸素的 \\(O(|X|)\\) 枚舉而已，但是在能夠 \\(O(1)\\) 取得 substring hash value 的狀況下，能夠加速成 \\(O(log |X|)\\)，具體而言，可以參照下列虛擬碼。

```C++
long long get_first_difference(X, Y) {
    int l = 0, r = X.size(), mid;
    while (l < r) {
        mid = (l + r) / 2
        if (substring_hash(X, 0, mid) == substring_hash(Y, 0, mid)) {
            l = mid;
        } else {
            r = mid;
        }
    }
    return l + 1
}
```

完成 1. 之後，只需要 \\(O(1)\\) 比較 \\(X[i] > Y[i]\\) 即可。

## Rolling Hash 字串匹配

> 給定你兩個字串 \\(T,P\\)，問你 \\(P\\) 在 \\(T\\) 中出現過幾次？

這個問題，可以使用 KMP/Z 演算法來解決，但是，也可以使用 Rolling-Hash 乾淨俐落的解決，在此，我們介紹 Rolling Hash 的解法。

藉由 Rolling Hash 前綴和，我們先對 \\(T\\) 做預處理，再來，我們枚舉 \\(i\\)，並查詢 \\(T[i,i+|P|]\\) 的值，如果 \\(H(P)\\) 等同 \\(H(T[i,i+|P|])\\)，我們就得知這兩串字串相等，也就能夠把這題做完了！只需要 \\(O(|T|+|P|)\\) 的時間複雜度，便能做完這一題！

```C++
long long string_match(string T, string P) {
    int occurences = 0;
    vector<long long> H = preprocess_substring(T);
    long long hash = polynomial_hash(P);
    for (int i = 0; i < T.size() - P.size(); i++) {
        if(substring_hash(H, i, i + len(P)) == hash)
            occurences += 1
    }   
    return occurences
}
```

乾淨、簡單、俐落！這就是 Rolling Hash 魅力！用短短幾行程式碼，即可解決問題！

## Rolling Hash 最大回文子字串

> 給定一個字串 \\(T\\)，已知最長的回文子字串長度是奇數，最長的回文子字串有多長？

這個問題，可以使用 Manacher 演算法來解決，但是，也可以使用 Rolling-Hash 乾淨俐落的解決，在此，我們介紹 Rolling Hash 的解法。

定義 \\(T\\) 倒過來的字串叫做 \\(T^\prime\\)，也就是說 \\(T^\prime = T[::-1]\\)

枚舉 \\(i\\)，對於 \\(T[i]\\)，可以藉由 Rolling Hash 判斷 \\(T[i,i+K]\\) 是否等於 \\(T^\prime[i-K,i]\\)，並對 \\(K\\) 二分搜，即可找到最大的 \\(K\\) 使得 \\(T[i-K,i+K]\\) 為回文字串。

二分搜耗費 \\(O(\log T)\\) 的時間，而每次枚舉耗費 \\(O(T)\\) 的時間，共計需要 \\(O(T \log T)\\) 的時間，即可把問題解決！

```C++
int max_palindrome(string T) {
    int ans = 0;
    string T_prime = reverse(T);
    vector<long long> H = preprocess_substring(T), H_prime = preprocess_substring(T_prime);
    for i in range(len(T)) {
        int l = 0, r = T.size();
        while (l < r) {
            int mid = (l + r) / 2
            if (substring_hash(H, i - mid, i) == substring_hash(H_prime, i, i + mid)) {
                l = mid
            } else {
                r = mid
            }
        }
        ans = max(ans, l)
    }
    return ans
}
```

眼尖的讀者可能會發現，這套演算法只能解決答案是奇數的情況，如果答案是偶數的話，要怎麼辦呢？很簡單，如果答案是偶數，在每一個字元之間，安插一個垃圾符號（Dummy character），再運行演算法，即可得到解答！垃圾符號不能在原本的字串中出現，不然會影響原先的演算法。

舉例而言，\\(T=ACDEED\\)，垃圾符號是 \$，新的字串就會是 `A \$ C \$ D \$ E \$ E \$ D`。

經過演算法計算後得知，最長的回文子字串長度是 7，從兩個 E 中間的 \$ 開始擴張，直到 D 停止擴張，這就是最長的回文子字串。而 \\( \lceil {\frac{7}{2}} \rceil=4 \\)，因此，最長的回文子字串大小是 4！

## 總結

儘管 Rolling Hash 並不是一個完美的解法，但在實際競賽中很有用，因為實作難度較低，且算法本身不複雜。

請各位同學要熟記如何使用 Rolling Hash 解決「字串匹配」與「最大回文子字串」的方法，儘管有 100% 正確的方法能解決該問題，但算法本身較複雜，且通用性較低。

總而言之，Rolling Hash 是解決字串問題的常用利器，請各位同學要熟記！

## 資源

- [介紹 Rolling hash 的 Medium Blog](https://medium.com/@lawrence910426/rolling-hash-%E7%9A%84%E5%A5%87%E6%B7%AB%E6%8A%80%E5%B7%A7-24a54ea32d10)
- [Introduction to Rolling Hash (OI Wiki)](https://oi-wiki.org/string/hash/)
- [Rolling hash and interesting problems](https://codeforces.com/blog/entry/60445)
- [Good Indian tutorial of Rolling Hash](https://www.youtube.com/watch?v=BQ9E-2umSWc)
- [滾動雜湊 -- 知乎](https://zhuanlan.zhihu.com/p/67368838)
