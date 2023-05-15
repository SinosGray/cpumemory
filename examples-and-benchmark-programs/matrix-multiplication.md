# A.1 矩阵乘法

这是在 6.2.1 节的矩阵乘法的完整基准测试程序。有关使用的 intrinsic 函式的细节，请读者参阅 [Intel 的参考手册](https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html)。

```c
#include <stdlib.h>
#include <stdio.h>
#include <emmintrin.h>
#define N 1000
double res[N][N] __attribute__ ((aligned (64)));
double mul1[N][N] __attribute__ ((aligned (64)));
double mul2[N][N] __attribute__ ((aligned (64)));
#define SM (CLS / sizeof (double))

int
main (void)
{
  // ... Initialize mul1 and mul2
  int i, i2, j, j2, k, k2;
  double *restrict rres;
  double *restrict rmul1;
  double *restrict rmul2;
  for (i = 0; i < N; i += SM)
    for (j = 0; j < N; j += SM)
      for (k = 0; k < N; k += SM)
        for (i2 = 0, rres = &res[i][j], rmul1 = &mul1[i][k]; i2 < SM;
             ++i2, rres += N, rmul1 += N)
          {
            _mm_prefetch (&rmul1[8], _MM_HINT_NTA);
            for (k2 = 0, rmul2 = &mul2[k][j]; k2 < SM; ++k2, rmul2 += N)
              {
                __m128d m1d = _mm_load_sd (&rmul1[k2]);
                m1d = _mm_unpacklo_pd (m1d, m1d);
                for (j2 = 0; j2 < SM; j2 += 2)
                  {
                    __m128d m2 = _mm_load_pd (&rmul2[j2]);
                    __m128d r2 = _mm_load_pd (&rres[j2]);
                    _mm_store_pd (&rres[j2],
                                  _mm_add_pd (_mm_mul_pd (m2, m1d), r2));
                  }
              }
          }
  // ... use res matrix
  return 0;
}
```

回圈的结构跟 6.2.1 节的最终型态几乎完全相同。唯一的大改变是 `rmul1[k2]` 值的载入被拉出内部回圈了，因为我们必须建立一个拥有两个相同元素值的向量。这即是 `_mm_unpacklo_pd()` intrinsic 函式所为。

其余唯一值得注意的事情是，我们明确地对齐了三个阵列，以令我们预期会位在同个cache行的值真的在那里找到。

