---
layout: post
title: Placeholder substitution in the preprocessor
tags: C++ macros preprocessor
toc: true
---

I came up with a trick that vastly simplifies some usecases of the preprocessor, and want to show it off.

This doesn't change what can or can't be done with the preprocessor, but makes the macro syntax *much* cleaner in some cases.

<a href="https://gcc.godbolt.org/#z:OYLghAFBqd5QCxAYwPYBMCmBRdBLAF1QCcAaPECAMzwBtMA7AQwFtMQByARg9KtQYEAysib0QXACx8BBAKoBnTAAUAHpwAMvAFYTStJg1DIApACYAQuYukl9ZATwDKjdAGFUtAK4sGIABwAzKSuADJ4DJgAcj4ARpjEIIEA7MEADqgKhE4MHt6%2BAemZ2QLhkTEs8YkpwXaYDjlCBEzEBHk%2BfkG2mPaOAk0tBGXRcQlJqbbNre0FXQpTQxEjlWM1AJS2qF7EyOwcAPT7ANQixJ60R0SXCJhHsagERCxHhuiXxACeR15ZRte3LCYyDO/2ImAAdCYNABBKHQ/YAKiO0LuqDoCTSBgIt2AjASTD6DCON1oaQS4KOABEvJi8KJsQojiYAGwacHslkaI4CXZHfjEI6YIEIQX0NiCI4RJmspQARwA%2BpzSEcwZigRFgEc1bsEJ4sMRGQB3Qgigg3I60PDzUWYcUEBSQmGcuWK1mSxlMC1Wgjcqj/PkkZ6ciBMUixUjINYQdngqMxtacx3Q6HofCEsS0L5C5Ainq2xg%2BiAKLw5l6MzlMJWc2JV1mmVlrI6iInxLUtAs3LIAL0wbwgEOAFLwfsIR0ivcZVzQgiYUrQLEBCg2cKO/rS7cEncwHrBKttqAAbr2Xl4ngS6RmvhBYqeXj6WJkfTzbmSBXm7cqfrdgyG1gm3VcuI%2BsGTD/lyUpmrcmCqOuDBZAIaxJnCygGDqeoJIyEHmomHJuu20oaNgACy8p4JyRwQCQBGka64EAGIEUIAASADyABKAAq8pEdCbhsSxQgmAArBY8puCxlLYAA4tgUTCZS5FWoKzCxPQ6AbEchoIHSuYwa8k6oARLrCVYgR0SYZnySZZFCQprJJgAkn6nrapguq0PqLxpGSLSYXBeBYL6BHRuyYHKpBbZgpu27bkcLA/D6rZpGCuxYG8sRfJyxHyso5FFrqrSMAZ1HKLRiFwhxNy7vhYgKIZqBpOmFyue5%2Brlqy2V4PKLHKBxeUKAV2JElcnI0T1XGNmaBKCnpDBvFcDAPNpfzDmODxamcB4Bb2yoRPMQpvKgfqiD8GovESCRnMQSEypgaS0e6LxHEo67EASVFTQQYAcH5SiFelmAEIamCMDNsHwQwFIOT6zZ3C%2BG6Qd2O1rgjW47rc8zEHgZJpH2A4UsGpBgZchl7QkPqevOgLlU6MLYKorCYtuIArlwFLZeJkkyVEEArqu/OrhAqikFwUYfKQZhRl2pCBBspB8wLq4RD62VcEygSUkc2VmBZVgwgLCb6/z9OwegBks0bAvK0cqjq5rXC65KEqZRrRw64EFhOz6XZ20cgS63CK5mEOEocxJ0mybzluKyGpAQFwEsy3%2B15x9IQmkMyf7KocRzKKjMUerQtCoIaAYClTTB%2Bf674Fgo4U3F8%2BFgg%2BR5vEwp6oICjiiEXHxJor/MQET2fHEId0tO9iRNp3gL9wPWskQ7Inyb7JjJBY2trwp0cJh7Cta7NZskxbKKK9blbL7Zq/rwnbvKv7ySUsqNaX5rFlv%2Bv0hHOnRzMlvAe0xRIECk05rQ5haAvMS4duZR1PgPX815lzRwFkPJBcDFYIlVr7bKQgOJsUgcoCAqtDboN3nrdBJt9LH33quUBMMEAQIRJ6d%2BTIzBmErGw5UCJYirzYTWNhADYT60kBSHO0IiR4BYIQc8AggpZRIjxAAmhvKBURcHylCGxciVAzhBlZKeOgCh9jIAPGCKg4IECJn3uYQIWAaCRCOEolR4k1FcU0SFWMTIhJuBocbEinMI5RA0SxAA6hALxPjkHz2VMJSJ6D54UQHuYZkRxlQJKgAiM0Vo/yNliQPHOUQQZt1TCUZgFxgzxnIvEE6Sh/QCAxoDIKr0OwF00nQC4QETxnm7pePcLdexzwSYLOhrCUlpPnhkrJS4/wRN8fApJZg/6LOVPMdAIAQD9IgJk7SS4ji5O8XMmOIzknJJWQQNZGzDyYC2VMxsGlYmHJQfcg5UT4HynlAANWhPKaEbEpJCHefsuJCSSHWLMLYzA9jbgAEU5DQlCJAh20cbF2KWEcOiITKTyiENgUIDFN6AIFpjEsPohHoLXuQhJ1tVCCMOU47Aqj1HuMefza2sL4UBlQBARs7KEUUrBAQbYRIMVYpxXi8ENK96P0OaC15Rwc6ULmubQ5OdrbJP4Fy3JiymTrwFUKii2zskSoDtKuVqqJTHMWRq7l084LAW1fywG%2Bqbk7MQpKqwpr4n83NfapZzJrVar9TqiweriBElWeszZhqlzGqldvL1q4fW2utCcq1qBNXJt9ck4Nobw3nKOJGq5LqjXuv/si6Ve9CVCVEccFEjAfBtLNE9TGZ1pxHgNDkc6h1Go5DEEcA8YgvDbkGawiFULHHQgANIMtknIIiRDlTvl2Q8uV9bnjIAMAoRk2Bg2zLlQLMOXNI5LtIMqLB9L5QOSiA5DiBCiEkTMN1XqGwgUCxzh4LwHkXi0Dqt8Wp8j5Rq21k%2Bvqbp8ApQIJmZU4CjBnQigoD4M5bYjVZLHd%2BkhJZWLlf/D2e6E3PXOesltfwOKoCaFjIwRDBRAsORSvDQz%2BYKGNAQUs/Zcnr0gQEmBJ7oOV1uNgdZqsQB7kFWGyBuD8HZUIcQ3W7G34vPw2W8l4LUUOIvVem9HjGzvK%2BSB95vNXY6e%2Bb8/5gLA7Rx4tO%2BUs751a3lnK2OXANAkPSbEFzEzkAS2c/vEhCJ9iBxU5CtFh7AlFkwAqFZd15TKhjIidcjwEgMD842EL3NglhMixF56UXlQQCMz8v5AL5R/jhGIi0JcEgAFp6BHguO2iGSYc50TwAaSD4tLhMAANaxU5AwPKxcGpHCwI1BAk1DLFm8iQGGGZfrbTkayIQdF5R0XYtZ3iTFyIDbSA6Uro96gCHQDE1k8XsRhoevFa0rYxBgiYOgL4zTopIz7KOZs31Erwy3ceeIQMQZEkBMCTImkbhEi8AwB7iM8A9nUkhQL47UuyXSxABgmXovZfusqE7iXitUmwBxaEDlQjWf8dAhHDlJJRA4jexR4TFtiWhBxCAtOVtsTW24JipAGBRkknjgnROoFHqCSxOQHFsAs4sBJRRypuf48J/DwXwvRfYpF8oZUtOohyFCKEXL2ViK9Ul2j1HmOzsaRdHc3bXid0hLet5M6mB20fCyStIknJpe87l5e8nlOOLU8qQBQy0FTb%2Bk5DxPiLEjg5XlFp8iRvIbmdhLDtFrvZf%2BPp0LUgHxGw0vBTYj4AWx2J9xzLvnbh6eBDT%2BLLsmebE5%2Br%2BCrsefVO3CT8X%2Bnkhy%2BkGloaKv2fwW5574Eev/fDQN6Cw45vHMpIWBvQCrgWnIF4MUdZgAGsoaEUQsWr7YrJIQ97PnGY4hoPLe%2BCumeK6ChPY/C9u/8ZP6fgGQNz%2Bygv5fq/185V%2Bdv3funn/08P/lkzRWJWToF%2BTeV%2ByeYkt%2BHEAKZgj%2BJEz%2B2AK%2Ba%2BG%2BH%2BaiX%2B%2B%2Bs%2B/%2BhWgK5%2B%2Bel%2BPO4BbgkB0BD%2B8Y8%2BbEi%2BCBr%2ByBW%2BqB2U3%2B5BPyHEGBx%2BAB2BI%2B464%2BN%2BU%2BUB8oZepBT%2BDBlBSB7%2BNBO%2BdB%2B%2BMBmBp%2BQB8euBoB%2BBxeRBvBJBoUZBFBiBb%2Bm%2Bn%2B4h8oP%2BHEkhLBWBZ%2B7BBeChE%2B3BAKbe/BcBghGh1B2hJEumpeR%2BThRhMhKKo%2B8hRe5hd%2BkgKhniAh6hVBIhDhx%2BehZeUhgBOBjeOOZhXBd%2BQksBuhthwRWhtBjh%2B%2BbekRbBwBchsR3h8RPBQk/hKWNhQRwhaRYhGRyRi%2BreLhxmbh0Rnh%2BR1%2BEBFh8ozISR8BdhIR6RLBHEiR2RxhuRMRnBbRd%2BzIJRahL%2BFRKBVRYRDB9Ogxhh0hTRHBYBih7RyQXRKRsxohaBjBnRQx7hIBLRBBShyQUxgRMxmhcxBxehRxKxURJheBBR4xPB/gOx5Rtx%2BxOh9O2xxxaxphbxhB7R/gVxZRNx9hfR9BtRHEAJTxORshoxGxPhPBoQaRc%2BsCHh6xcR7xAKbEmJpBQJrxrRbgoQ0IQgQgDkdEi%2BoQDkuCc%2BYx5JlJ1JtJGiDJXE0IygygoQ1OzJFJVJNJdJnJ8o3O2AbgXEOK0K4eiJxWLxXhZJgpbJIp6i3JvJPuqhAprJwpHJapPJfJkexxeyCpZxxeypup9J%2BpGpkepB2pQp7JVpXEbOEpk6io/eQxpp9pKpepzpTErpYpqJ/iFpjpop4pkp2Aoq2AMp0pZpHMIZqpXEuu3u2Jpx3plpopLpbgbp6ZoZ6i4ZUp0ZRwXycZwZOpeZSZREeuqZeRuZiZgZIukp2K0ZTJQZYkCZvpDZEpIuUZ0K9RJ%2BgBXpbZLJDp9ZyZmpniQ5eJXGCOvEXEh%2BqoP60WpBnpIxzRzJJOQSc5gGEAi5Cg0WLApApBLAEAG5AuHuskXuiie4mI%2B52mcpU5IJm58o4ulI1OyOh2dpw5z5r5i%2Bh%2ByOpZ/OgSF5FOVOQ2yoxxj5ZJP5Eu8oh%2BX505MFb5O5kFa5uJT555v5O5yOh5x5/ZrB8paFwJ0F55ZOl5VOc%2Bq5yJ653555QuIuYuEuSO3QaQVAoOyA8oL08oMey5WptFwF9FiuWF/5Uu/FaWZFoF3uaObFDAHFXFPFEFcpJJipBBz5gljFyF8FfFiFdFCuGli%2BzBrh0hUFqlulDFL5sFs%2ByOL0MlHFLF0WClpBNl7FkeXFjYtOJeDOTOq22A62HOXOYlCOv5olOlwFuC2AKuJwS26umuuWZ5wF25GguWgJGOBIp2DAhF1F6FJFAlelSuEVzFzlslnFUWjl2lGFuV5l4VpUh%2B8V4lnuYFRVclpVaVWOylgFM58uVVyucFrZoVaW6l%2BVpUhlDRxlRFpJpllViu1VO5TV0WXFpAPFjYu%2B3RqRcxkW6ObYCWxuJlxez51VzFn55VOVaWM1tVgVQSElV5Q27VdVCOZ1fVFVp1PVI1A5Zm41Kle155M1s%2BuFqhp5F1PyTZt8qFWVxF4Bsk0IFgoQDK06iiIS7EWKIe/EQgs%2BqgjYYxkN0NsN2A8NiN3EvEKNbemNUQUNMN8ocNCNbESNhNAkMB6N3KIVbxWN5NlN%2BNyNAkfhkkdETNrRLNONeN1NBNoeQgfhcg6%2B2APNt1bZ/NFNuNVNNNItMBCFzNpN2NctgtitKNgQ7p/s4KVFOJ4NfOstbNQtHNQgOtNqCcNKH1gFJt8t7NtNFtvVjYSVNtYNE1xtatrNDtZtTtOts%2BjY1tu1Ou3tAtCtwtRNj1fNYdGtEd5tyxRlg5ttJNZN4djtItXNktQsVeKdMtsdptWtnN8o4t3N0dENBdvtRdQgnR2UhdkdAk0teJ9tmtDdQgiRKtMdadcdGdKNxR%2BFbhIdJELd8dTt/dwsFeR5qhXYOdQ91mldrd5tnR6NdtC9o9Itkxs9edzda9vdAkm9ndFd3d9d5tlxteOtcp3Kc9I9e9QglxVt19u9ftItlxh%2Brtj9x9VdbdlxgdRwSKHtn1odn9i9TtlxddX95tNqGgH96tzE7EXE5taNGN%2Bd3dcDnEbdxNKDsDrE6D5t9NUYGkqd2D8DGDgZPNq9qDODCDTtYtEtUtMD5NaD1DSt5dXtlDJD5tOt59BtaZWDjDVDbdltQdpA7thtntQDxDuD/tLtRwbtDDDKTDgjO5wjojvDO97DUjmdrDEj/DHDY9A9Y1ADFDkjzDRNZDOdNs8j2KAj5ttDZdh9bDJjbdtdJEij5tTdqtGjpjAkHdx1R9TjCdRpSlVjbjY9keE9HeU9niM96NITNjTty9yD6jATCT4THjXdKTG9tpfjjjujmjKNZ9Hpl96T/jeT3jd9keQdcTejL9Mj0D29njmTBTyjf91T%2BTAkYDrj8TItUDj9lIlFD5Hprgc9lRL5U6DKYxozUkbEUNFEaCYjgDJEozFg4zDZbxUzMzFgG8b8DTrRyzqzHN2jSzcxYzVmHNQTSd71RjkzJzKzZztN2TniNzohpzDK3DD5uz4BGzUNRzvRAK0zUN2zBjgB9m1zbZozUQLEy2DkbEjJDjUmJzkL0LsLXEwLVzCzgFELULdEML6iaeK5HzYLeJ0pjBWlniQxkC45NZKJxL0ZyLcLqhFLOuVZKZc9Emc%2BNiPDtZbZq1exn%2B8LkJQhPx2%2BFzo1ydRLbxvLwraijzSTkrux0rAK7zlzionzfOUr0JAKYxGrfzUJkZTIar1xQrmrazrROrozV1YFc95rJzlrKZArNRUJurxr%2BrXLMRWCAV05ShCcppqsIGeyANXr7RXAIGoLGL2sAbzJShZgYbpxwG40kbw50bobvrJEgQibQbd%2BwQqbyhCbnrIJShOt40sbeR2UkgGbBb7R0gObfhebgblbvhKbtt2UQkFbZJSh6cObxRdbUb7R3bvUJb7rJEKS%2Bb7b7RzIg7zR2UkxPbSb47TbRj2UyQbb5xWxk7cOJElxs7mbPBW7A7Ob/gK7mxd%2B/g67wWJE4J27DbHxC74bKeDOE9crZJqewsRwGeObnlZeETleHVzhr77WleH7dR37pAXev7HEbe/7yo0smk7V8Z5ZiZvzI5PpTpaLmVd7UCREPEc%2BZ7DizL1ZpBObN9z9KNSOrAmAT7pT6dJHAks%2B4DIDItutNizAbAcHw9sdoTPTLHFHxjZTbddHXTNTKNTH4K3HbH89/TDrsk/ToNGHTElJx%2BCb/9GHdrE595KrObqnPyAzGnzbJEWnFgOnYr7ymnDV3uYkRnb1qri7%2BnZni%2BknjLhLKnAKMNUkvEi%2BREQgHybgOUW%2Byg/Ebg2AVJ7EsjmnLn0k7n3EXnPn/nEVAXQXQgq2LJUQUkl6REN69ODkLEUQoXenGiRJ/1ObhCxJ9bY7d%2BGJKBpAxxpXq7d%2BhJlXOb9XNBc%2B4nJLqeeAUTpRdODOOCdL9O5AUYsncbrjfXB%2BvzbXB%2BspKrJpeXE3s%2B1hzZ0KpLu%2B45U3xn6Hw3i3jBytqhvXS39OAnROeua3VnrXo3fBu3I3%2B3%2BhK3LL15Q3pbV3jBVhl3W3zh%2BHUlD3Q7b3AxSRE3beH393wTs3o3nRC3E3iRgPJ3BFZ3132x4PoPt3x3X3U7T3OLKL43dL6PuC0PRhM3NnW3ehEAHXC3nlu%2BJLehA3aHsPjrv%2BXx3UUQhplR1PObFPixTBf3dLehh%2B1xQgk6DkpUzPgJrPXP7PO3ART3ehh3z%2BfPAvfz1PNP4RnPS3ehMBvP/PgvdxwvIPKv7PL3EvhP7PZe6vcvQvSliv7PvjBvbPcJAPkJsvmv%2Bx2vBPNvhxyvtPv3Jvjvn%2BzvGHrv/x7vDxK1DBDv8vvvm3nHKNBrLveC43sfKPG72KiiRE4uhOSBHLiyJys3yfqf3UbEHLyQPihfObNr%2BxDrpfn%2B2rCrJrqnuPhjGHz%2B2XTPdxCPS32PDOzzIrdzDKhznfMrALns4fj3jrofzPDrWL9LHf4LtzBztNNXfO3zg/5vObThDOJPr3ZPfxHEVPCf57/Rh%2BwspBqjw/q/s%2BaQ1tnXljeXq/MB5/i1Mbh/qhx/33q/Zed/aQMbaQwQj/niz/qP/Rbed/p/3SDSAf%2BudAnqv0SJADFqIAxaunDAFX8IB%2B%2BTotAK/6LVpAaQdOGkAnYIC/%2BifVftsVQGwDMBi1CdmkGSAiNL%2BeAvfrplWwTdiel/DmHUWyhnoyyo5X0tTwgr9dJQg3YHkgLz6G84SDA0nswJIisD2yCHDgccQgqU8eBLPa/t8m574sn%2BK/RQezzRqX98s40fTDoSUFD8X%2BaguEvTU0GKdeoOg6olLwV6qCPeX7EwbQLMGR5dBYvKwQoI96Qc7B3ybQY4IsFG8XB/AvQokQQFaCHB9xPXn4Iw6wk3eQQ0wVxG8ELE4SiddbicRP6GDGC2xaIfYNiGhC4SjxFVskJiIOQqyhOaUszh%2BRRBIWeOEXG9yxK8C8hhQ3kot1KFr4Kh9OBRnS3IQYt6hxQ6Mk0PKEsRKhbQ/bhJ2j6dCihjQ1bM0P6GtCtuhnUgtymp5dDxhLOSYQMLe6mlFhJQiYX0NWEktRI0nEYacQ2E9CthLQqoSS2k4WMjh0KXoacMGESc0mEAbzLbSuE3CphZwultJzSaWMPS0ndYWMM2Es4yej7I4C8KS705wm6edqqCMBHgjy83ePWoEFzzPD/hxwlnMzggCHk64VAUgIdmABeBtojYaESBSvLhIiRpQqSHIDIo/IQRKI64atgq40E4oKyZUNiPApHA8Rc2KEbSNKHM5D8mIvgDiNIAcisAhI7katlU6H4aRDQgEctlWwUiyKGgakUSIZGyQmRz0FkcqEOzsj8RgULkdKNRGyi2Iv1WwAKNxE6ieORI1TrPilHdC6RaIuUZSMkghsUQyotImqLrh8hNRyoYUbcD1G2ieR7EGAvyOxFmiCRNopYcSKpwwFwxMo8kY6OwCPoXRYolnCqJy6Hl1RnotkT6Px6jD9Rdow0WXmDGCifRoovMaUNU5l4YxBouMWRR1pJiyx9It0emI9GsitR2Yv0RGOZxt4ixoYkUVWPzGqc28/YgMSznlGSQ/C9Y/0Y2JQLuiNRWY80TmMOHJjDRiRHsUKPNGlipxLOVTokWHGrYaxkkYopOIjGpjZxmYtsQuI7Gxj2InRNcSWL3Hbi7OnRB8YaPlBjjsAkxY8TKNPHNi5xF4zkX8IbH2i2I2xO8RuJfGqdtiL4g8dgEuJfiDRP45keeO9GXjAJW418e%2BMVHQgMRJokMSEE3ERiyeZIh0QqJ%2BSo4KOOEjMa2KvHViSJkkLCZHh7EETrxo4%2BMYqMM6/jkJaEzsXROwAMS%2BRuEnEY2CoDxxNRzEg0TFVCDUtmixE1iaRI4mCSjqniE8rfC1HGlZJb4tidp04msiFAoktkdIO4ksTNJZFZ0ThKxGCiLRy4oicuJgnOjyJUYHSWJKMm0S5JTosieZNNHiT8xdkl8pRJbFeiXJPk3ifZIEkWT1IfICADGxuovjJJ0k8dBpPfEhsFJ4Uy/ieWilqSlK0EkKdpKQm6SopXouviCyCkjiTJkkRMZ5LwlWSgJ3XCAIlPjGJiHJ/kv8TROCluSExHkpidlPamPoFJVEwKciJqkwTGpYUryZFOCAxSiRcUkqfuN4m9TKpgovChNMyl5DupZUjqdhKcnPQIAy0xSnkNw63B6ptY3KalOqnoSbJQ03iXWKalbT1IM04CetOukLSIpR0ySDrT6kBTwK90jCfGOumjSQxwkiAF/BenLjppg09CTBPenPS0pQMwqepNslXTcp/U7acDL2lJCDpa098ROOhlnTCJqeV6dgAnE3S8pzk8GTxPalEznp3k0qVjL8m3SMZBMomf9KEmRSf4IMmqWDKMaMy6ZikmGWzLRmndMZ8Y7GVtL0n8yipgKBmQjPalHicZ1MsEQzgJlHjiZyMu6WTOMnvjlZVMoWWRWKIfS/xUsy6TLLInMyIpIkidpNNBka4pJ30mCXrOhl4ULZK0pITrMPFIzPpekp2QLJh6GyIZvEz8XLJfEXS/Z7Uz8SrM%2BlqyuZ0s9aWHO1kEzJi%2BsrierNckxyTZ2skSRQMtkczrZ8UtFPHJ5mpS8Kmc52YLPzmbSSZ204ud7LcK%2BzyZ60uCYHKJHBy6574uCeHJam2zeJbcuOdHNbk8zVZtcjWfGLbmmzAZp7LOehM5m5iQ59cguaaJhnjyS5MPV2bBPdlzi9Ji86udIUHkpz3xJ0%2BebjJlHNyh5VI75IukckVzI508lufGM6mKT5ZD0veYnNbE7y2p60tOV1JElJUJ5EYqeUuKNnvyUp88vCt/KXluEV5a8zMXpNAVbzipycgcXZ0s4EVvpEopBUYRQV2d5ujnXTlHJqmqdxe6ndbhgvIre4Lu5LJzv/PQmDi0Fp%2BYhZJUURW9CFVnOhVeTB7YKiF8C8sXZ3h7sLmFnC6cYyNzkOIppOcwjvwpZy4JWhh1S/ugG%2BkcQoWe87CQSzqHLj5FgCuRQotvmzDeFMPIkWosUWmlzm0pOQLJEC6vimI8oA4XkQ8ovtIRUVbrhCLfbtUbFqLYEVngRFIijG3lFnL5TZzQy5QD8w0c1OQkvj9Ft8%2BCfmJWHTDzh6%2BOqcuKiXvD9u2WWUDMmcVLZShvipiKbO6ApLupAkgeaEs0WnyXxCSu4RcKJGlK3uyS1JaaW8Ws4mIxo1KQEu6nGiCleiopZJCVHxLth0Sj4bEoqU9LEl1IgJWbltp1LMlQY3mc0tkmTK2lqijpQygiWvCdhfS/pgMtuFVKRli46xekp8rrZCxUy8LIEuZwHK5lNUsJcUvWVvCyl/S7pRsra7VLRlXi3ZT4vWzdjDluS2Se8rOXoSLlnSpZScOuVbdyldyoFQ8q2VpLXxmS1cR8uOXsQYVPyiMX8sWUlLBlNytZaCpWVJKIVtSl5fUtvGwrupBKxFTKORVdKaplSmJRiopVorNlRyp5Ri3GXrZQJhK2SSypJUGiyVAK5YbSqpVxKaV9yvro8u2UxFacPEJfIGV6hMQjg/gXFfKDiliLnlSuKRaQSJGSKRcaHImKaQjwtd5%2BaJFzmkSq5KU9VhRAkoavlTHA3AhgN7H%2Bm/AdQlmSoOKAlE0hYxsQ1wJSOBnqBtYYceRe/Iim1UhttBiKEDNqsfSQIdYtteUI%2BiDXxteo2qnWpAn9iRqi2ZgxNSGsjV%2BFIEkgbVbW1TVlt01RjeUMUUgRCRtV/bLiCWoLUYsOili6dtqpnZ5rh2Va04vKEuKQJkg2qvdhWqXbNrfV4JSBLKsjWXtG18oYdX1EAQ5xKsU66dTOtnVzq48NiCIBui8CBQLIbgYjMAAsjYAF1pxSzDOnVzzpsAi6MUMugUwCw10TYTdNul3Qrp8McufsCetPTh4Q2E6ZRAyg0y3odVUavTC%2Bnoz8wcMnsW9USkIwgAN1RwUjORg1BUYeOQG/9Rxlg0DwmMhAVjDBo4z3qeMTYPjFrEEyLxhMuacTHgnDylQ8sIbMhHJj/WrglMGLdTNegZzKLPBIQlhLJxhB7r56c6cJEev3iOYnh6CRBFxs8yBAeNawDgBsFoCcAhIvAPwNwF4CoBOAPiSwNYGehbAdg34cFDwFIAEBNAImjYJ1hACSBq0AATkkDJBUgBm/wMyHBSSANAGgYIGJo4CSBJNWm0gLJo4C8AFAIAJKppo4BaANgcAWADAEQAoBO4aQdEGQAoAQB5wIW%2BgIkA3SGBgAXAAzU5j4B0BTs7m68E5tiARAWgHwTgOpsy3MBPgLEWINoC9W5beAVMAsCxAYCZgnNWAG8MACtVFx3N0mnEbaDi3iBvNvAT1Q0CPDNatAIQVQPUFPB7B%2BtysHoE5stCxA3onwDwFgDK0aasYLAebe2nuBKBKQbWowJaCMBaaNgVAAwMAAUAfI8AmAQ0CxDJBSb1N/AQQCIDEDsApAMgQQIoBUDqBOtpAXQDGwMDbbTACmywPoDwCxB3NkADYA1EJDNbKsqyd%2BN9qsCWAzAXISrCEgzBHB4d0EAgG9CR1kh0AhgbuJVkugkAFAMm9tFjCwCA7GadQBoM4H7BzQZgfgBOGECWAVAqgegDIFkEJDU6mdxQQkMMAZ1jAE4ZOwkAMGmCeAOgegPnY0AWBc7RgiQXnQsDZ3S7BgEulYFLo2B1RtguwCQKJvE2ObXtLmm2OZsqzMhy2sWv4AlvBBswuQRCNMFRBsSixeAXmnzRsBuC3YxgjNXTUJBEQaAhIbCVkFwBSDMghIBmgzRQLs0ObSAS2oSElSk39aXNbmjzRpp21xwAtEAJAN5HICUAjdwASrPyC7hObewhAEgAFD0A0BaAp2SrI7rahOartwgHuHdukCV6ntagJze9oW0gwC9zmfQJro5xOaXNygVUGcF2BboqILEU8GkFvCqA9dBuy9XFr/oGbTd4Ic3bgDz0ChrdyoDwCwCi0JBR0ksW3Ttod0HRndGu%2BzbwDD0R6u9nAGPZ5p32kBXd7uz3Ysg0A%2B7kgfugPUHs4CBAtdUes/XHs61Zw7NZgd/TJs/127tNpAerDkD01AA%3D" class="button">Demo on Godbolt</a> &nbsp; <a href="https://github.com/HolyBlackCat/macros/blob/master/include/em/macros/utils/codegen.h" class="button">Library on Github</a>

[The basic syntax](#the-basic-syntax) \| [Example applications](#example-applications) \| [Comparison with other methods](#comparison-with-other-methods) \| [How does it work?](#how-does-it-work)

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

* `_P_` is a macro that must be added before parentheses if an `_i_` placeholder appears inside of them. So `EM_STR(_1_)` is spelled here as `EM_STR _P_(_1_)`.

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

## How does it work?

The basic idea is that the `_i_` placeholders get replaced with `)(foo,`.

So `int _1_ = _2_;` becomes something like `int )(foo, = )(bar, ;`. If we parenthesize this, it becomes a list of the form `(...)(...)(...)`, which we can then loop over using [`SF_FOR_EACH(...)`](#looping-with-a-library) or any other similar library.

`EM_CODEGEN((a)(b)(c), sep, pattern)` uses two nested loops. The outer loop iterates over `(a)(b)(c)`, and then the inner loop iterates over the pattern (as shown above) and substitutes the values.

Astute readers might have noticed that `#define _i_ )(foo,` isn't usable inside of parentheses. E.g. after `f(_1_)` expands to `( f( )(,foo ) )`, we have no way of skipping `f` and recursing into `( )(,foo )`. That's why the `_P_` macro (that was explained [here](#generating-enums-with-string-conversion-functions)) is needed. `_P_(...)` flattens the parentheses, expanding to `)(open, ... )(close,`. Then the same loop that expands placeholders can revert those back to `(`,`)`.

&nbsp;

That's all for today!
