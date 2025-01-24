Binomial coefficient represent the number of ways to choose '**k**' objects out a set of '**n**' distinct objects, where the order of selection does not matter, and repetitions are not allowed.

$${n \choose k} = \frac{n!}{k!(n-k)!}$$
Let's take an example to understand this better. Suppose we have a set of 5 distinct objects: A, B, C, D, and E. We want to choose 3 objects from this set. The number of ways to do this is given by:

${5 \choose 3} = \frac{5!}{3!(5-3)!} = \frac{5\times4\times3\times2\times1}{(3\times2\times1)(2\times1)} = \frac{120}{(6)(2)} = 10$

Therefore, there are 10 ways to choose 3 objects from a set of 5 distinct objects.

Implementation in C++

```c++
#include <cstring> // for memset
#include <iostream>

using namespace std;

int binomialCoefficients(int n, int k) {
   int C[k + 1];
   memset(C, 0, sizeof(C));
   C[0] = 1;
   for (int i = 1; i <= n; i++) {
      for (int j = min(i, k); j > 0; j--)
         C[j] = C[j] + C[j-1];
   }
   return C[k];
}

int main() {
   int n=8, k=5;
   cout<<"The value of C("<<n<<", "<<k<<") is "<<binomialCoefficients(n,k);
   return 0;
}
```
