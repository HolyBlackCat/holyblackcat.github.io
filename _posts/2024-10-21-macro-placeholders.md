---
layout: post
title: Placeholder substitution in the preprocessor
tags: C++ macros preprocessor
toc: true
---

I came up with a trick that vastly simplifies some usecases of the preprocessor, and want to show it off.

This doesn't change what can or can't be done with the preprocessor, but makes the macro syntax *much* cleaner in some cases.

<a href="https://gcc.godbolt.org/#z:OYLghAFBqd5QCxAYwPYBMCmBRdBLAF1QCcAaPECAMzwBtMA7AQwFtMQByARg9KtQYEAysib0QXACx8BBAKoBnTAAUAHpwAMvAFYTStJg1DIApACYAQuYukl9ZATwDKjdAGFUtAK4sGIM6SuADJ4DJgAcj4ARpjEIADMAOwAnKQADqgKhE4MHt6%2B/umZ2QIhYZEsMXFJqXaYDjlCBEzEBHk%2BfgF1DQJNLQRlEdGxCSm2za3tBV0TA6FDlSM1AJS2qF7EyOwcAPQ7ANQixJ60%2B0RnCJj7UagERCz7huhnxACe%2B15ZRhdXLEzIxx%2BxEwADoTBoAILgiE7ABU%2Bwh11QdFiaQMBCuwEYsSYjgE%2B0utDSsRB%2BwAIl40XhRBiFPsTAA2DQglmMjT7ARbfb8Yj7TD/BB8%2BhsQT7UL0plKACOAH02aR9sC0f9QsB9sqtghPFhiHSAO6EQUES77Wh4BQEIWYEUEBRgyFs6VypliulMU3my2oKg/bkkB5siBMUhRUjIZYQFkgiNR5Zs%2B0QiHofB45i0WjvfnIQWYYWMS0QBRebOPOlspjytlRStM0xM5b7UQMa5XNItfOXLIAL0wzwgoOApLwPsI%2BzCvbp5zQgiY4rQLD%2BClW0P2vrbwMEncwbuBiutqAAbr3Hl57rjqWIM/sIFFT49LSxMl6GFzibzc9b8wrPldA0HlnGLrnFilqBkwgHsuKxpXJgqhtgwWQCMsCbQsoBiatqsR0lBJrxqyLrthKGjYAAsjKeBsteJBEeRzqQQAYkRQgABIAPIAEoACoyiREJuOxrFCCYACsFgym4rFktgADi2DhCJZKUeafLMFE9DoKs%2Bx6gg1I5nBTyTqgRFOiJVjxPRJjmQppkUcJilMgmACSPruhqmBarQOqPGkxItNhCF4FgHI%2BoGsbymu7abtu277CwnyWjE6rAlsWDPFE7xsqRMrKJRhZaq0jCGTRyh0ch0KcZcu6EWIChGagaSpmI6roe5mG6kRWV4DKrHKJxuUKPlGLNucbK0T13ENsauJ8vpDDPOcDC3Dp3zDmOtxJYegW9gqoQWvyzzeo2TCfKqjzNrExzEChkqYGkdGuo8%2BxKOuuLUVNBBgBw/lKAVaWYAQeqYIwM3wYhDCko5lpNi26qRdB3bbRFG7wzFhEWsQeDEmkfYDqSgakBBZxGbtsSWu685/GVDqQtgqisGi24gCuXCkllElSbJ4QQCuq686uECqKQXARq8pBmBGXakPEqykDzfOrqElpZVw9LxGS%2BxZWYllWJCfNxrrvO0/B6CGUzBt84r%2ByqKr6tcNrYqihlav7Fr8QWA7lpdjb%2BzxNr0IrmYQ6imzkkyXJ3Pm/LQakBAXBi1LAE3jH0jCaQDIAQqez7MocNbm66aoHqfq8hTTD%2Bb6H42goCrQe8hHAo%2BR7PMdZ6OKI6avAm8u8xABOZwcQi3S0r1xI2qALkwXfdxrZF26JCneyYiQWJrS%2BKZHcZu3LGuzSbRNm4i8uWxW892Yvy9xy7Cq%2B4kZIKtWp/q5ZT/L9I%2Byp/sDJr371OIvEpLTgtI2BALQZ7iVDpzCOh9u7/hvMuSOfNe7wOgfLWEytvZZSEJxdiYDlAQGVvrFBm8dYoKNgZfe29VyAKhiA3ksJ3TP3pGYMwFZmEKlhFERezDqzMJ/lCXWkhSRZwhM2PALBCDnnxIdTKZFeIAE0V7gPCFgmUQR2KUSoMcAMTJTx0AUDsZAB5gRUBBAgeM29zDxCwDQMI%2Bx5GKIkso7iajIwsgbCJNwlDDZkXZmHcIqjWIAHUID0mEp4hB08FQeK8TA7u5gGT7AVNPa8EBYTGnNABdxYTu5Z3CEDZuyYShpiIq46MlEYiiB/L6AQVwlBeh9OuDs0V9R0FOCBE8rcLwdz3I3XsU9kn82oUwhJSTp5QDSTpJcAFQnhJQckkJ8t4nxIVBadAIAQC9NSekpc%2BwsmzIGT3IZSyzAMhWQQNZGzDyYC2ZMhsmlokRJgfcsJMSo4yhlAANQhDKCE7FpJCHeXs15hCLFmCsZgGxVwACKcgIRBDAXbSOljrHzH2PRQJZIZRCGwEERiq9f583RsWS0/CUFLxIcky2qg%2BGvPsdgJRKiXGvItqKGFcK/SoAgA2Nl8LyXAgIBsZs6LMXYtxSCalW9b7Av9o8/YWcyFzVNq8rOlt4n8E5e4k59Jl78sFdeCZGTxV%2BylbKlVoojknPVVyseCFQJar5f9PVNzDUSqsCauZvMzV2q/pa1AGrhnxO1RYXVxBmyrPWZsg1S4jWSvXh61cXqbVAOOQyK1DYLU%2BoSQ6gVoanrnP2BGq5zro2uu/kiqVW8CXCSEQcREjAfBaUNI9dGp1pxHl1DkM6B0Go5CageMQXhtz9KYeCyFdiIQAGl6VyTkCRfBCpK47IefGlSDbkAGAUHSbAQaZnMu8eAjm4dF2kAVOgulMpHLhEcpxXB%2BCyJmG6r1VYey%2BZZw8F4TyjxaC1Q%2BEoDqs8wEPvGpRfAyUCAZgVNmQwwBTrQSeq8Gc1sRpMmjs/SQ4tzGyu/m7Xdsrw0gBbd8TiqAmgYyMPgvkQKsPL1wyuwlBoCAln7O4mjIdD1c2PZBsuVxsDrOViAPcObmyYOwbegh2sWNPxedRuNfNkUQtReey917SkNneV8x93EZTc2dup75vz/mAplSg3iU6ZQzrnRrWWsro5cA0IQ%2BZUQHNjOQGLez29CGwh2P7MFKLbFsb8YWTAsoVm3RlAqKMcI2x3FiAwLzDYAucwCcE0LIWnphYVBAPTPy/kAplABaEwjTSF1iAAWnoEeU47awYJizvRPAupwOizOEwAA1jFNkDBcq0D9WkfYWAGoIEmkZIsPkSBQ0vN9LawUmL0RlPRDi5m%2BLMUoj1%2BqdpCsD3qAIdAUSmTRYxKG%2B6cUgGJTEMCJg6B3iNKigjPso4myfQSq2MuSg/oAyBs2P4AJMhaUuM2LwDAbvwzwD2DSKFfMKf8z4iBclksQAYKl8L6W7oKgO7F/L5JsCcQhI5II5mYfsYvVJcInFr1yJCUIObbgIScQgFT%2Bbi3sDLdIAwCMUkcd44Jwevx3U5CcWwOxGUFhJJyIVBz3H%2BPEtw9YvzwXWKBfKAVAz8IcgghBEy1lUivUxco%2BR%2Bjo7mknR3M26E7dgTiBMB8qdTA7bXjpJWs2NkEuufS/8Y5EnZPOIU7CkBIysFja%2BjZLxfirF9jZW0779kBvwbGfk2Ol3UufG04FqQV4DZqVgssa8Hzo7UWJ%2B5zTzi8RU%2Biy7Bnyx2fK9gq7LnvzVwC9s1p5IUvpBJZ6gr1nsFOeu/xFr73vUdeocN%2Bx5Lwv0kLDXoBVwVTYDsFyPMwADWUBCcImKV/sTkkIO9nz9OcQ0Fl3fOXDP5ZBZDhPo/Xc%2BIn1PmUXBNOz6yvPpfK%2B1/ZV%2BVvnfGnn%2B04P9lgzeWBWDo5%2B%2Bel%2BSe4kN%2BnEAKZgj%2BZEz%2B2Ay%2Bq%2B6%2BH%2ByiX%2Be%2BM%2B/%2BuWgKZ%2Beetije1%2Bk%2BUBMoQGvUsBMo8BiBb%2BG%2Bn%2BWU3%2B7EC%2BtOGBR%2BAB2BQ%2BF%2BnO4BbgkBAKJesYc%2B9BL%2BSB7%2Bm%2BqBtBe%2BMBmBJ%2BQBUIIBeBYB4%2BhBPBD%2BfBT%2BAhCBr%2ByBIh2%2BYh5BAhtOEhzBWBp%2BbBoBHBCht%2BLeKhcBahlBmhNBZEGmtOJekhgBOB9eWOZhbM3BMokgyhbi/BC%2B6hQh1BohDh3yP%2Bxeh%2BjhRh0h8ephY%2BXhihMowkZBFBGhwh9hzBnELeLhrBwBuBI%2BnhBBt%2Bwkfh0YARghVBKB2hYRuhDB2RUR%2BmMRbhw%2BHhCRxRRBDIqRNh6RIRNRWRKRuRxh%2BR7h%2BBEBSRDIZRCW1hgRthGRoRR%2BERgxhhUhLR7B7R4xt%2BiQ3RsxvR1RaBPynEXRQxsRshhRGxXBSRiQUxFRQRVRWhBxERxxKxrhJhchRRmxRBAAHDsZUXYQsY4ZxNsScWsfEVfp8QCl8TcaobscEfsToREcCS8XkTIQUW0eCZcbfkECEbPlAnEe8Rcd4exDiXwaCQSRiUEBCEIEII5PRAvkEI5FgrPmMW4JSdSbSfSYydxBCMoMoEEBTiyWyTSXSaolyTKBztgG4NxNilCuHsiflm8ecRSVScKZySojyXyT7v4YKSqRyaKeqbyfydpicbsoqeiZwUKXqQyQaZqZHtqfIWzJaSKdadxG4MxJKROnKL3kMWaTqeyc6WKW6R6eKQ6T4k6WqdxBKVKdgCKtgLKTKeaYXuGfqdxNrt7niWcYmY6bqQGSokGW4J6X6aqSmSGQLlKVinGfsF8lmWGTmRGQTjrhmWiUWVaWKVGdKXGcyaGeJMmS6aWZKQLrGVCo0cfoAb6d2ayXWSWWmVqeUeOR8b4klnxNxAfkqN%2BuFnwT6SMa0SybDv4suXfhAGuQoOFiwKQHwSwBALuUTh7nJF7nInuGiCeWpvKfORcXucLqLgjqQLtnwdebziLmSAvgfojjWTzklreaTuTv1gqCcW%2BRiR%2BYBcBV2QuYhaLoeXBduesQhUTkhYeYjmeReSOSwQqVhWCZwR%2BZBfebPluaiaMROR%2BbLgLkLkhd%2Bc9FQIDsgDKM9DKDHhufaahUTkxfLnhSBeLgxTeZ7tBexZxdxWFnxbBfKWSUqRRUJXLixehQfn%2BRJbzsJRpUBRhUpfBapbpepZ%2BQZTPojjJS%2BOFjxekLiIdgwPxeUdZVxUFndA2AzkXvTnNgtkLszm6azuzjpUlkheJYJbzlgtgErocHNqrurplv%2BUueWRoJliCWjg5RjspWBYuTLmZVFXglZbdBxTZTxQpdpRFUlnpQrtFTKAfklXDlRdJcVbJWVZlYbsZYXoxflYrnVShe%2BWpcxTVSVEwdEVIZ1W7nzkNQVYea5bZRlnxQ2DvmkXCVoaFqjrDDFh1WReSSZUlgVd%2Bb%2BQJQNZFb1fVSFY1VJd7v1tlQ1f4jNVpcdThadbVaNU0eNTtSpV1UTjNTPoRf4VeRdfueWZfJhXRTud2XJBCBYEEPSlOnIoEhxJiiHgJEIDPqoA2GMVDTDXDdgAjUjTxHxKjS3ljeENDbDTKPDYjexMjUTYJDARjVyuFRsdjRTVTQTSjYJL4VJPRMzeCazbjfjTTYTaHkIL4XIGvtgLzbdZDWTTjZTXjdTbTaLTARVSzXLWzYrRzXTUIPEF6b7GCrRfiV9VrhrYLUrSLajXrdanHNSp9WBQLQrULcrVbX1Q2KlXbeDdheAY7ezcLZzbrYeQ2LbRNWRL7Vrf7TrZYU9T7WbU7RbQHcsWNWOfbaTeTebdraLdzVLQLBXqnbLenfHZncTTKBLTzf1fzXHX7S7YJF0VlNXZbYJDLR8eHc7Y3UICkWrZXYXQ3YncaUZfnS3VXRHTXR3dpoLGXuef4V2LnaHeZsPW3QHV0RjQ7QvQnTrZMbPYPerT3SPe3ZvV3bHbvYvTrdcdXnrfKVynPa3evaLdcTbdfWvcXYJNcQfu7Y/cfbfajdcTPsHR/fLb3afdzoA6LdahoP/RTSxBxNxAHejZjQXfLVA1xO3STQg5A2xMgwHQzRGJpGnYgxgzA1HSGbzavYXUg4Q1naXZLdLRA/SuQ%2B3arTHdzo7fQwHXrefUbZmXg%2Bg9A%2B3dbcHaQJ7cbaQ/g7w2w27fsB7bQ1igQ3w0HfsCHdvd3aI5g1HRXUfSoxQ6jUne9SnV7eRcw3Haw0Q%2BXSvUI1w2g3Q7IwHeLdQ%2Bo4Y2Q9YxvTvsY6Lc3TvZo%2B3Z3Uw6bY42IzraUcRTEdI649o%2BPWnm3lPeUTPRjSE046LcvfA0PX46owk%2BPe48ozw6k6jQfT42HUY/E9/frZYkMVfUoxo1k1oy/dpn/eUw454wHa/WA3E/43ffI4ivo7tfU5U%2B3dcZgoU4JM03U6bWSDRa%2Bd6a4HPX0cLpOvSmMdM9JOxNDdeMgsI/M9UTM2Zus1oTKIs9DSvE/MM2RNMxYLM43fY1lCc2c5zf3cnUZkc/MQCqc2Zjcxc8cxs88/Shw6%2BQ8ws0sxYG8487s/8wc0E1IdZp0ybe8zs%2BEKxPNo5OxEyYfdztM7C/C4i9xGC3o2s92ai3C/RAiyoqnpuT85C2BTKYcY9eUUMWAjOU2fRR8RSwSxi2M3czKLSyRI2XPVguxLPiU6Szix8StfcZ/sizCX8UC1i/c2S2McK/8QCoC3K0C982y3PUq9M7Kz0atVvn8TGfSA8%2Bqxs5q7CSK8osTnedBWq1q6awCk1emWKzMRK9M3cVJOrJw2iegsFQud4XHGacrJprsoDd60kffuNBC8I5rIGyyd4QEH6/egG164SUkSQZxOG5mVlPEFGxOd4fEGmx62RHreNFm8G7foW71Hm%2B4VlJIMW0mxYRW60VWwm0G7W0Qb4WG3G8kTWxid4anB26UUW4m920kf2%2BWx2wkoO5wd4acmO029GxMZpvW2OllIkF25O1cYu6isu7O9m1cQux218au%2BYd8Ru9DjKFCQO820O7fue6O/bU3nThPUkxcSnoLPsOnh20XiXhPW3k%2BxiU4anm%2BwqOXh%2B83q3u3r%2B5waB6%2B81pLFpNldmf6RGYC5OYhymVK6RWSyHCRLxLPiezxmHZy/a/4R2zfc/dvswGwBB90xnZHaLTPvXXvQHcU2ChR5gPB/kyk1U%2BR6wGxyIz07Ay4wMwCufax%2Bx/PaMw6%2BJ%2Bh2J8xFSUfkWx0xG2RHa7OS%2Baq3e8p1dQway7o%2B8h2yp8Ljp6OXpxp%2Ba1Bd7uJEZyRfp1p%2BKVZ0YfpwCrDdJHxAviREIB8m4NlJvsoAJG4NgNSRxJI456ojJK5zxB515759FX5wF0IItpOeENJBeiRNerTo5KxOEMF6Z9iSgbiR23gqSZe2u1iSEaQCccV0ewCsSSgXh2AjVyIbPmJxSynngFE9MeJCnpgnGYceQBGGDUpxWVCpS4Cy1/vnKWy6aaZ2NzPlYUNyN1roRw%2BQN%2Bm2RGN4w%2BUd18N4wTvjORN7pxh4N2N7wf4Vt4cTAYtzrvt8Z4d6t/N6B3N8d7t0t9dyRc1z17Tt45t2tx9w0Zd9dSt/m/d0cWQWNykf98twPZhz99t0CaD793XQR1d4D5WzD%2Bi0i3k/N8y1gq90YVN9D/NxERAG13N95WdxEX19Jx2xSxEVSx18/pl0aX0VT9Nz1xEbN6d2j7T8tQIUIBOo5CVMzyCdT2z3oZxBtx1zT2L/R463zwL489J%2B98NxESd994T2Lxd7L/z4L/scL6z8r2L9HWr1L/USXuK3Lzrw8XrwTyb4cV95L6L/US3ub9rwr9b0d474cV0Y95759zzwvhb270pUr3UYcdsT7wb/UYj6H4H0L8H9T0J/qzb9gqNynyjw22t3IiRCLvjkgXyyckstN1nzn91Ly3wUvJ4okLMoN4aw8ZJ7X5/sa06xsyp7jx9QTwz%2BEEz/sRH%2Bj3Ttszq58%2BcwP2a3s%2B7O73d8/rHz35j3i335V5cx89c3TQv9Czq2P2368aZ4CcT%2B12AmT7UbTpT%2Bn0u4f/vsS/4eY0D4CTPmkLbXv1f6j1kTAXf%2BkAEILHwY/xn1kSXq/2kAEGkFzYf9L%2BHbQEi3j/4ADc2aQaQMAPKJf9T%2BAxCABAPSBQDpAaQVOLALzoE9ASXRZAYAPSBoDU4aQU5JgKtigC982xPAagPSBEDTkaQRIIIwf7kDS%2BwPXfqT1A5ZRT0tZVDi6Wk6wUj%2BYofrlD0G4aZFstvVrnv3vbO8yIXAnslOV4EnFYKFPQQSz2wHhExeB%2BUgfAM3a1F2eF/alvJ16jvJHiGgxXswIiIM09%2B2WcaMYIRLS8zB2/dQab30FqdvkNg7THYPqIGE2Wpxa/k4MOIt5SB1gowR4N0Fi9nC8fRwaHz95BDDBWmEwU7wcFqDohIPWIaIJCEJC7eSQkQf4NpzbE0hbgjIZ4K97ZDMyjkTlvjhlJ%2BUfk4QWFjjgFz3dcSQgnweUL5JDdqhq%2BOobTjoY9cSEwjVoZULjIdDahrEeoT0O27ick%2B/Qioe0MWydDRh3Q%2BbgCz4JcppOAw2YULnmFjD7uZpdYVULmEjDthFLMSHJEOZks9hQwg4V0IaEUtThudfYBcKhTDDrh4w8TukwgDuZ7ajw54QsJuE9dTh6TMgd6VOG7CZh%2BwoXN5UfYPCwRlwiEbTnCbvsvhMIp4Ql3hGl5O8BteIDniRFtDwRjOXlmeWrhUAfypAYAF4C2gNhvhi2FTiEipFC5qh0kOQLeR%2BTQjcRsI1RCEVigrIFQxImCvsDJHTZsqdI/EQfkJF8ASRAorAJSORHVCVOB%2BVkYMJRH0jFsjI28hoBZHCjcuIhLkU9B5EKhds/I8kUFCFEyjFsflP6rYHFG7ZJRvHYUSpxnwKiNh%2BI3ZkyKkj35EQmozkWeV1Hch9RCoG0fj2mFsilR%2BImAmKOJHWijRto00ULhU4wFHReIhka6OwAPoPRMYjkSgR1HVxfRfIgMSaODHVC/KJecMRKKjHSiCx1IrTiXgTHsikxt5PWmmIrFC4tRckLMXqNzFRjAxZQ9MX5RbwljIxFImsSGJU4t4hxhYlUcmN8KNjFR1QlsVl29HZjeRBovMaCKbH4iUi/Y0kWWLHGViLW3uFIjuOVFC5VRUkUotOKdFzi2xOY5cZ2PzEzizRHELopuJtHlj7xsYrTl0UPHOiTx2ASYueLxGXiFx7Ym8YKNXFvj8R2xZ8duLtFadtiX4usVJGuL/j2RgE7kdeP9G3iwJTohCdgHVEQgIAm46MWuO8rCicJeE5HGxwIloSlxd47CROLVE/JtM/Y18XROPHJj1RALICehKwmJj6JUkciaKMtERiGwVAWOPqJYl4j4qQQelq0VIl8TcJhnTcURUvgGiTScktiQxPwlcTeRCgMSXyMUE8Tax8k90VRKJEkiiJ4EkiT2OMmMSF0EYbSeJMMkhicJ7opiUJJ/ISSjJGkt0YpOol%2BinJ447ydgFcmCSzJGkbkBAACA3UvxUkmSWOnUkujby9%2BTie5KOrlFLyUU1SUpXgk2StJfkp6JFL9Gb9AUdXBKT%2BNTGmSrRgQTySGKslricJqYiifZPykaQApD4oKY1MqnCScpHU3yT6KXGlTrJvU75KFKtEiSIAubaKcKNiltSjxiUqSA%2BhSlhS9%2Bl5SaVlJ8E9T5pKYn5FRP6krIJpRUgyTiPAk4SGxXU8yTVOqF1STp8khsU1N2mLjHJx01iVtLunnTwpZU5MXrRSl7SYKs078V9MYmjTupokt%2BB9PTEzTnpvEoKd9PekrSIAYMxShtM%2Bn1idpDkgqYjOKn5ZBp9U%2BSVOLhkWSnR10l6T%2BKnH3T0ZrUqGV5K2lkz3pl09qTTL6mPS/pVM5yXjKBl0yIpH8cGWuMhnnChpjMpaVVKIrcykZB3emXNNJloz8puk0WVjNWbdjcZQUs8QTIlmdc6cKM08bZMowPTgJtE6GVtLPFuSwpasnCaUR%2BnMzds/0s2RzOYkRTTkU0iGWrmknWz5J5suGURQdnrTxZm0n8SrPRm6SvZYsm7grLRKazfx0s5aYTLxHEyDZP4v8eTJan6zqZ8cxiZzPDmTELZwEnGTdKCkJzgZHkiKQwMdm8znZcU1FBnN8nLSiKxc72SHN9nJi/xusnMbpNrnBy3uOckmcmKQmqyvxsclOd3O1mUSKZyctmUFKQnGyxpDc28tcSzncTWZgUraRPILnhTRJXxcSTFLLmuzx5Vc4Wf4UvLrz9J2U8OT3IDkQBD5dcjudvK2mRyqp0c9kf3LHk3zvkdk5uTROvk/i05ds8OUzOzkfzkxts1KeNNSolzwJfMoMbnJvlCyIx8MkBZfJiLTypI0s36bpLgXtyYincvETSJJbqd%2BZa4uUfZxPz/T7RhCvLMQq04S8sZ5CvcXIlV6uCbu1C8znIiN70LrOC83cUwvt5UL2F74mhd738Lut3CMEmheHwEUCtFZ4Ey8eXNsTTSy55fHhQrm6GHU9%2B6Af6ZxDhafz8JOC8WcKPUVQK1FGigBcsLEUtD0xeizRWaRuYyk5Ackfzs6OYjsszSXlF9mnk8rU40R0HbKs4sxZQjM8mI7EWSwZzVCAqzEOGdKFNkcQ35RU3RYYuZHISQxWwxYbcLXwQBhRiSv4dt3SxShpkXi3ykzmWwrzbAwWCJexEEm/SDRMS5%2BV%2BPSWvC7haSw4Uko%2B5ZKclTivJf5WWwWjlp4SnqRaPKUKhKlmi6pQ0oyVSd6lLw%2B7s0pNz20gl%2BSt0mGNSlFLslPU%2BZX0q/HmKAF8Sn4UcP%2BEpKxlvw14YiHCVTLAlbSpbG6WLELLul6ki5asoGUbKhl4y5JaMz2XbLMlRyrsWiRmXtK3SfYy5cUp6m/LblZi2JUgs2VXD9l83OpemJqUTL3luS50SEo3F/Kll6kpFUCrXHrK4lDyiFU8tSXQrhlByyZR8vcJfKzlzEJ8cipKUUr0V4EzFaCuxWvLRl%2BKx5U0rhWtKEVy2SCZSp6lcqaVToulfSjBWbCCVkK3ZcypxWsrilxy4Rgzl4iL4QyvUZiPsC%2BLsrYp8ik5YooFyz5hRWCJRUMQJhmk787LZWIatDZGCEUmmQ1Q%2BkAxWrNMxq%2BNsBntoyg9aYCX2E6rLbcRXVlqp1b4TASSBDVbbc1Y20dVktki9q5IoapHaeqsoUaw1ZMTARfwnVkxGwQmu9WhrriYCRIIauuIpqt2Ia4Rme3DUqqnVN7aNWRFLXGYs4pWatTWtrV1r61cee2qZmnSq4502ABdMKCXTSYUE9aB4Oulewawd0y6ZJG7n7CdqT04ee/OOgUT0plMN6CPFlhTbYyqMZKCtO7GHW8x8MhGNUMRlIyqgKMvHDdauHJS0ZkkCgBjExkPWsZCcgWTjEdD/S8YQA/GQTHqhEw4IF1Rq4hJJlPW8wy0ZLJTFejpzaK7V2mRhAN0hDNr56s6EJO2u3i2ZPhKCOBHBtczxAENywDgKsFoCcBhIvAPwNwF4CoBOAniSwNYCejrBNgv4MFDwFIAEBNAGG1YK1hADCRBEXxDQBoESBfF4grGqQPEAZDJB15WGjgJIFw10bSAhGjgLwAUAgBUqtGjgFoFWBwBYAMARACgHHhpAUQZACgBAHnDqb6AcQftUYC4DJA7MfAOgIdik03hRNUQUIC0FeCcBqN1m5gG8FYhRBtA9QWTdRopj5hWIDADMKJqwC3hgAbgSbPZt4BYA/gRgcQHJrC0NZ3NeAI8FJui2BBVA9QU8NsC0DkBBAuYUTWaCiCW43gHgLAKFpo0YwWAxW9tDcCUBkhrQ0GM0EYDo2rAqABgYAAoA%2BR4BMAeoViMSDw3Ub%2BAggEQGIHYBSAZAggRQCoHUBJbdAAQAwPVtMAkbLA%2BgPAFECk2QBVg9UVMIltKyrJn4c2qwJYDMDshSsgSS8PsCO2wQCAluU7cSHQCGA24pWC6CQAUAEb20GMLACtqZrdBUwLgOaFMD8Bxxgg8wCoFUD0AZAsgqYX7SDuKCphBgQOkYHHE%2B2NBZgEO%2BHR%2BB6AMA%2BgrQGHcMDiDw6kdngDoHoAtD9AsdiwHHasFqgbAtgEgTDdhpE1JbxNVsL4gyFKwMhq2BmtUEZpBAsx2Q%2BCFMNREsTCxeAsm%2BTasEuCXYRgTNRjXZlSqCbhNpAMrcJFSp4aMt4myTdJpo0NaY4ymiAEgB8jkBKA7O0rDyD%2BDC7eAvYQgCQECh6AaAtAQ7KVjF06hEtGWvrcIHbhDbpALusbWoFE1TaStQMK3fZn0C07Wcom8TcoCVDHAtgm6aiKxFPBpA7wqgJnSzrZ0zaOdyQLnSCB524ALdvIAXQqA8AsBdNsQEdOLCF0NbRd%2B0CXTTqE28AFdSu0PZwDV0yby9pARjZIGrRGbkgZgLgAyCM1fFhIDIVDbxqD0cB4gdOlXY3o13RaM4gmswOPoI2T7TdGcarDkBACSAgAA" class="button">Demo on Godbolt</a> &nbsp; <a href="https://github.com/HolyBlackCat/macros/blob/master/include/em/macros/utils/codegen.h" class="button">Library on Github</a>

## The basic syntax

The macro I'm presenting takes a piece of source code with placeholders and pastes it multiple times, substituting different placeholder values.

For example, given `int _1_ = _2_;` and tuples `(a,10), (b,20), (c,30)` it produces:
```cpp
int a = 10;
int b = 20;
int c = 30;
```
The exact syntax in this case is:
```cpp
EM_CODEGEN((a,10)(b,20)(c,30),, int _1_ = _2_;)
```

While this might look like a basic feature to have, the C/C++ preprocessor is very limited, so implementing this isn't trivial.

The applications are endless! For example:

* [Generating enums with string conversion functions.](#generating-enums-with-string-conversion-functions)
* [Generating reflection information for classes.](#generating-reflection-information-for-classes)
* [Generating `const`/`&`/`&&`-qualified function overloads.](#generating-const-qualified-function-overloads)
* [Generating repetitive classes](#generating-repetitive-vec2vec3vec4-classes), such as `vec2`/`vec3`/`vec4` (which was the original motivation for this library).

While all this was already possible before, this library makes it much easier.


## Example applications

### Generating enums with string conversion functions

```cpp
#define MAKE_ENUM(E, elems) \
    enum class E { \
        EM_CODEGEN(elems,, _1_ = _2_,) \
    }; \
    std::string ToString(E e) \
    { \
        switch (e) { EM_CODEGEN(elems,, case E::EM_1: return EM_STR _P_(_1_);) } \
    }

MAKE_ENUM( E,
    (a,10)
    (b,20)
    (c,30)
)
```
This expands to:
```cpp
enum class E
{
    a = 10,
    b = 20,
    c = 30,
};
std::string ToString(E e)
{
    switch (e)
    {
        case E::a: return "a";
        case E::b: return "b";
        case E::c: return "c";
    }
}
```
Where:

* `EM_STR` is the classical stringification macro, such that `EM_STR(foo)` expands to `"foo"`:

  ```cpp
  #define EM_STR(...) EM_STR_(__VA_ARGS__)
  #define EM_STR_(...) #__VA_ARGS__
  ```

* `_P_` is a macro that must be added pefore parentheses if an `_i_` placeholder appears inside of them. So `EM_STR(_1_)` is spelled here as `EM_STR _P_(_1_)`.

While [magic_enum](https://github.com/Neargye/magic_enum) is a thing (and can magically convert between enums and strings without any macros), it has its limitations (slow compilation times, inability to customize the names of the constants, not detecting large enum values, etc), so preprocessor-based solutions are still relevant.

The macro above doesn't support omitting the values of the enum elements, but adding support for that is easy:
```cpp
#define MAKE_ENUM(E, elems) \
    enum class E { \
        EM_CODEGEN(elems,, _1_ _2_OPT_,) \
    }; \
    std::string ToString(E e) \
    { \
        switch (e) { EM_CODEGEN(elems,, case E::EM_1: return EM_STR _P_(_1_);) } \
    }

MAKE_ENUM( E,
    (a,=10)
    (b)
    (c,=30)
)
```
Here `_2_OPT_` is a version of `_2_` that expands to nothing if the value isn't specified, instead of erroring out.

And here's a version with a nicer call syntax without `=`:
```cpp
#define MAKE_ENUM(E, elems) \
    enum class E { \
        EM_CODEGEN(elems,, _1_ MAYBE_INIT _P_(_2_OPT_),) \
    }; \
    std::string ToString(E e) \
    { \
        switch (e) { EM_CODEGEN(elems,, case E::EM_1: return EM_STR _P_(_1_);) } \
    }
#define MAYBE_INIT(...) __VA_OPT__(= __VA_ARGS__)

MAKE_ENUM( E,
    (a,10)
    (b)
    (c,30)
)
```

### Generating reflection information for classes

```cpp
#define REFLECTED(m) \
    EM_CODEGEN(m,, _1_ _2_ _3_OPT_;) \
    auto members() {return std::tie( EM_CODEGEN(m, (,), _2_) );} \
    auto member_names() {return std::array{ EM_CODEGEN(m, (,), EM_STR _P_(_2_)) };}

struct A
{
    REFLECTED(
        (int,a)
        (float,b,=12.3f)
    )
};
```
This expands to:
```cpp
struct A {
    int a;
    float b = 12.3f;
    auto members() {return std::tie(a, b);}
    auto member_names() {return std::array{"a", "b"};}
};
```
The new piece of syntax here is the second argument of `EM_CODEGEN`, which acts as an optional separator inserted between expansions (a comma in this case).

Astute readers might notice that this macro chokes on e.g. array types: `(int[3],a)` produces `int[3] a;`, which isn't legal. Luckily this can be easily fixed by replacing `_1_` with `std::type_identity_t<_1_>`.

Another problem is that types with embedded commas need special care: `(std::map<int,float>,a)` would be incorrectly parsed as <code>(std::map<int, &nbsp;float>, &nbsp;a)</code>. The type needs to be parenthesized: `((std::map<int,float>), a)`. `EM_CODEGEN` automatically removes parentheses if present, so this will produce `std::map<int,float> a;` (not `(std::map<int,float>) a;`, which would be illegal).

### Generating `const`/`&`/`&&`-qualified function overloads

```cpp
#define MAYBE_CONST_LR(...) \
    EM_CODEGEN_LOW( \
        , \
        (      & ,         ((*this))) \
        (const & ,         ((*this))) \
        (      &&, std::move(*this) ) \
        (const &&, std::move(*this) ),, \
        (__VA_ARGS__) \
    )

#define QUAL _1_
#define FWD_SELF _2_

struct A
{
    int x;

    MAYBE_CONST_LR(
        int QUAL foo() QUAL {return FWD_SELF.x;}
    )
};
```
This expands to:
```cpp
struct A
{
    std::string x;

    int &foo() & { return (*this).x; }
    int const &foo() const & { return (*this).x; }
    int &&foo() && { return std::move(*this).x; }
    int const &&foo() const && { return std::move(*this).x; }
};
```

This is useful for generating methods such as `std::optional`'s `.value()`.

While C++23 does add a feature (explicit object parameters, aka "deducing this") that makes this possible without macros, the feature [has its limitations](https://stackoverflow.com/q/79081096/2752075), meaning it's not always viable (or is this is a compiler bug?).

The new piece of syntax here is `EM_CODEGEN_LOW(...)`. This version of the macro is necessary when the code with placeholders is passed as a macro parameter. (A piece of source code with placeholders is toxic and can't be passed to another macro without parenthesizing it, since `_i_` expand to `)(...`, which otherwise break macro expansion. Unlike `EM_CODEGEN`, `EM_CODEGEN_LOW` assumes the code is already parenthesized, and that's it.)

Here we're using `((*this))` instead of `(*this)` because, again, one set of parentheses is automatically removed from the element if present. In this case it's undesirable (`*this.x` isn't what we want), so we add another pair of parentheses (one pair is removed, leaving us with `(*this).x`).

### Generating repetitive `vec2`/`vec3`/`vec4` classes

This was the original motivation for this library. Those small fixed-size vectors are commonly used in graphics.

```cpp
template <typename T, int N> struct vec;

#define MAKE_VEC(m) \
    template <typename T> \
    struct vec<T, EM_CODEGEN(m,, +1)> \
    { \
        EM_CODEGEN(m,, T _1_{};) \
        vec() {} \
        vec(EM_CODEGEN(m,(,), T _1_)) : EM_CODEGEN(m,(,), _1_ _P_(std::move _P_(_1_))) {} \
    };

MAKE_VEC((x)(y))
MAKE_VEC((x)(y)(z))
MAKE_VEC((x)(y)(z)(w))
#undef MAKE_VEC
```
This expands to:
```cpp
template <typename T>
struct vec<T, +1+1>
{
    T x{};
    T y{};

    vec() {}
    vec(T x, T y) : x(std::move(x)), y(std::move(y)) {}
};

template <typename T>
struct vec<T, +1+1+1>
{
    T x{};
    T y{};
    T z{};

    vec() {}
    vec(T x, T y, T z) : x(std::move(x)), y(std::move(y)), z(std::move(z)) {}
};

template <typename T>
struct vec<T, +1+1+1+1>
{
    T x{};
    T y{};
    T z{};
    T w{};

    vec() {}
    vec(T x, T y, T z, T w) : x(std::move(x)), y(std::move(y)), z(std::move(z)), w(std::move(w)) {}
};
```

## Comparison with other methods

Let's see how this library compares with some alternative approaches.

Here's how else our ['generating enums with string conversions'](#generating-enums-with-string-conversion-functions) example can be implemented:

### X-macro

The classic solution is an [X-macro](https://en.wikipedia.org/wiki/X_macro):
```cpp
// Shared implementation:
#define MAKE_ENUM(E, F) \
    enum class E { F(ELEM) }; \
    inline std::string ToString(E e) \
    { \
        using type = E; \
        switch (e) { F(CASE) } \
    }
#define ELEM(name, value) name = value,
#define CASE(name, value) case type::name: return #name;
```
```cpp
// Declaring a single enum:
#define E_MEMBERS(X) \
    X(e1,10) \
    X(e2,20) \
    X(e3,30)
MAKE_ENUM(E, E_MEMBERS)
```
After expansion:
```cpp
enum class E
{
    e1 = 10,
    e2 = 20,
    e3 = 30,
};

inline std::string ToString(E e)
{
    using type = E;
    switch (e)
    {
        case type::e1: return "e1";
        case type::e2: return "e2";
        case type::e3: return "e3";
    }
}
```

This is trivial to implement, but the enum definition syntax doesn't look good (requires a separate `#define` per enum).

The implementation of `MAKE_ENUM` requires multiple `#define`s, and the loop bodies (`ELEM`, `CASE`) have to be defined out of line, which harms readability if many loops are needed.

Plus, as you might have noticed, we can't use the enum name in `CASE(...)` (hence the `type` typedef), and in general can't pass any outside information into those loops, without making the X-lists (`E_MEMBERS` in this case) uglier:
```cpp
#define E_MEMBERS(X,Y) \
    X(e1,10,Y) \
    X(e2,20,Y) \
    X(e3,30,Y)
```
### Looping manually

The [additional `#define` per call](#x-macro) is undesirable, it would be great to simplify the syntax to something along the lines of:

```cpp
MAKE_ENUM( E,
    (e1,10)
    (e2,20)
    (e3,30)
)
```
But how do we loop over the elements? It can be done manually, without any supporting libraries:

```cpp
// Shared implementation:
#define MAKE_ENUM(E, elems) \
    enum class E { END(ELEM_A elems) }; \
    inline std::string ToString(E e) \
    { \
        using type = E; \
        switch (e) { END(CASE_A elems) } \
    }

#define END(...) END_(__VA_ARGS__)
#define END_(...) __VA_ARGS__##_END

#define ELEM(name, value) name = value,
#define ELEM_A(...) ELEM(__VA_ARGS__) ELEM_B
#define ELEM_B(...) ELEM(__VA_ARGS__) ELEM_A
#define ELEM_A_END
#define ELEM_B_END

#define CASE(name, value) case type::name: return #name;
#define CASE_A(...) CASE(__VA_ARGS__) CASE_B
#define CASE_B(...) CASE(__VA_ARGS__) CASE_A
#define CASE_A_END
#define CASE_B_END
```
After expansion, same as before:
```cpp
enum class E
{
    e1 = 10,
    e2 = 20,
    e3 = 30,
};

inline std::string ToString(E e)
{
    using type = E;
    switch (e)
    {
        case type::e1: return "e1";
        case type::e2: return "e2";
        case type::e3: return "e3";
    }
}
```

While the call syntax looks clean now, the implementaiton is ugly. Not only do we need 5 `#define`s worth of boilerplate per loop (and the loop bodies are still defined out of line), we also have no way of smuggling external informaton into the loop body (`ELEM`, `CASE`), meaning the `using type = E;` workaround is unavoidable.

### Looping with a library

There are libraries that help you write preprocessor loops, such as Boost.PP with its [`BOOST_PP_SEQ_FOR_EACH(...)`](https://www.boost.org/doc/libs/1_86_0/libs/preprocessor/doc/ref/seq_for_each.html). Those usually come with a limit on the number of iterations they can perform (limited by the number of boilerplate macros they have pasted into their headers), but it's possible to support unlimited number of iterations ([I have a library for that](https://github.com/HolyBlackCat/macro_sequence_for), and it's used internally by the macro I'm presenting here).

I'm going to use my own library for this example (see the previous link for the explanation of the syntax), but all of them should be more or less similar.

```cpp
// Shared implementation:
#define MAKE_ENUM(E, elems) \
    enum class E { SF_FOR_EACH(ELEM, SF_NULL, SF_NULL,, elems) }; \
    inline std::string ToString(E e) \
    { \
        switch (e) { SF_FOR_EACH(CASE, SF_STATE, SF_NULL, E, elems) } \
    }

#define ELEM(n, E, name, value) name = value,
#define CASE(n, E, name, value) case E::name: return #name;
```
```cpp
// Declaring a single enum:
MAKE_ENUM( E,
    (e1,10)
    (e2,20)
    (e3,30)
)
```
After expansion:
```cpp
enum class E {
    e1 = 10,
    e2 = 20,
    e3 = 30,
};

inline std::string ToString(E e)
{
    switch (e)
    {
        case E::e1: return "e1";
        case E::e2: return "e2";
        case E::e3: return "e3";
    }
}
```
Now we have significantly less boilerplate per loop, and we *are* able to use the external information inside of the loop body (the enum name in `case E::e1:`, in this case).

But there's still an additional `#define` per loop, and the loop bodies are still defined out-of-line.

And the library I'm presenting fixes both. :P

### This library

To recap:

```cpp
#define MAKE_ENUM(E, elems) \
    enum class E { \
        EM_CODEGEN(elems,, _1_ = _2_,) \
    }; \
    std::string ToString(E e) \
    { \
        switch (e) { EM_CODEGEN(elems,, case E::EM_1: return EM_STR _P_(_1_);) } \
    }

MAKE_ENUM( E,
    (a,10)
    (b,20)
    (c,30)
)
```

This finally gives us 0 extra macros per loop, and the loop bodies are now inline.

&nbsp;

That's all!
