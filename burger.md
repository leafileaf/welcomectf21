# burger

As always, start off by looking at the provided source code:

```py
import hashlib

FLAG = b'???'

assert len(FLAG) == 56

block = 16

def hash(text):
    res = ''
    text = text + FLAG
    # padding
    text += b'\0' * (block - (len(text) % block))
    for i in range(0, len(text), block):
        sha512 = hashlib.sha512()
        sha512.update(text[i:i + block])
        res += sha512.hexdigest()[:block * 2]
    return res
```

Okay, so we get to input some text which gets concatenated with the flag.

The combined string is then chopped up into blocks of length 16, each one hashed separately, before they are combined again.

Crucially, the input text can be of arbitrary length!

I tried inputting 15 'A's:

```
Welcome to Burger hashing machine!!!
Please input your text in hexadecimal :
414141414141414141414141414141
Here's the hashed value :
272719bd2130afcbc64d5564704fb6624413fab0a6ff0f153fa10080134379c2d52e2ba9a6ecd5c8e85bd84368a356747bb1fe194df4c137e5497c6e38dfb7b40a62dfed732b7072464d0c195cdf207f
```

This is the result of hashing the string `AAAAAAAAAAAAAAA???...flag...???`.

The first 32 characters is what we're interested in, because we know 15 of the 16 characters used to produce this bit of the hash.

A bit of brute forcing should reveal the last character, which would be the first character of the flag:

```py
for i in range(255):
    if hash(("AAAAAAAAAAAAAAA" + chr(i)).encode("utf-8"))[:6] == "272719":
        print(chr(i))
        break
```

We quickly get the result, `g`.

Using the same principle, we should be able to acquire the second character of the flag by inputting 14 'A's, because we once again know 15 of the 16 characters used to produce the first 32 characters of that hash, and so on.

Thus, we proceed to get the hashed values when we input 14 through 0 'A's.

```
14: d993d9dbefea6a3d535c5f0d05bc1efce14f3be6b7b1d57f4135160ad0299c6bf93c7c1f6a2559124c9c4d4e7be0b7c4382c5142b30ea90c0ee14236c4ad10c041dbd56b3f2f4ac900aae1583bd9f338
13: c9833c11691ffacf5c8ffc9148685ed235932eba704eda1c014289aa120a820484adf07fce2f1a89baee36c78e4f80a3f41e56a24b441f1c4db28c5006586cb460cd63ea9d892b060d79ef95cdb7416a
12: 3b401138f7497268c7bb8e05063b810212cbaf75397a8907dbd9a6a4f41043f5457ed122d6e3835e1a0852bda663554686ead864b441a86c9bb58e12d08067f7ee977692b8245c4f300815d455b9cc47
11: c8ccdca9f33519efc2559365867636649011c6f71e0c8086f3ac4eb0c775b38350a2e7844773d387f487d778477f010e83e839f7947c2355031e62f12294f126130d4b65205c5fa79e1bf92612b28235
10: 7e77d9f7beddf597389535c602a11ef3aaacba5eb2b16d4e075071fa8490c9f795aaf77378db0d069533c125a1305bd736a6e47ce8add52e1d7036dd7e5d3aa07fc163c04fdcc4bc20420f94f6723392
9: 58cde488877a334a64134a307eb8d34fe284db3325de9c447468394942ca63359e90038a0de82ff55e2cb694aa099948f218718828a87bd48867ea4c6b903e5f102af0664473885f070e7e17ebd737ac
8: 0971ec74b10cda82eaa6a756fb813750ebf195ff8edce298a47f1794f844bdfabc32ce8be41ef50b5b21f7a864d7a1f3ec5151af3a49e6cf0bb926722e93dabf0b6cbac838dfe7f47ea1bd0df00ec282
7: c63295b4ba684c1741bd07666e87a058ff3e1e50a325360b9269f6998e748020743ae58bccee046569ead7fce3f072b3ae31c48a2fbf414cc65b5cab82c9d6c3
6: 414dc440ffe977bace8545f2a2e336a4cdc45516325c72b0053d57fad4518e07ab7a341014070f1f1ce78c906eb334a2df7249830a55b06000aef19fc2fc9ca3
5: 7d0eb4db3089fe7a927bfb073b5ed6df7eca06e57b613c3a300a103feca4111f529e1eec4f56f10bb953b5940f8055b4b10be46023b828750a2bf91dca6240c7
4: 65b71cecdfe2d1e302f27e60fa505edc96b2e0218359f690c13f4d8af7adcf7ba1d38c8f8159b29f557e2a995d902854c2cb6b55271b8abc3cbbc01214f997fe
3: 4f119c0040805456e7bf9d03e44c3f5b36ceac3dde19e4c718e8120781446aa84a305307ab4ae8b94d2565b0a4746c7869477912114615f1e2b1b07c4ad8f876
2: f4bc23ca0fe64509d8e055b5820c8e8ca137ebb1af3e658a06b443fe5113a21c90e7fed74be6ed77c52113a8ae7b53c5ea5fe6905e8e895f1f71012786c253bc
1: 2a530974b3c6969ea998ff45d3c0690b7c8745885b8da399b80fcc8a2ac383968ca30dbfc513da3bd720758cf35d784bfc59fedd3343a9e7dd577124f992c099
0: 27c18b28ecfda28db2987f05757ecf768dd3bdd0a9d7a58de25fca29343fdd753415b1d6ddabeefddaf1e3d14c6e278b5f19bc9e66fcbef5f5001bfb25d923db
```

Well, the flag is 56 characters long. We could use a little bit of automation to help us along.

We split these (including the first hash with 15 'A's) into 32-character long "target" strings, and put them all into a python list:

```py
targets = ['272719bd2130afcbc64d5564704fb662',
           'd993d9dbefea6a3d535c5f0d05bc1efc',
           'c9833c11691ffacf5c8ffc9148685ed2',
           '3b401138f7497268c7bb8e05063b8102',
           'c8ccdca9f33519efc255936586763664',
           '7e77d9f7beddf597389535c602a11ef3',
           '58cde488877a334a64134a307eb8d34f',
           '0971ec74b10cda82eaa6a756fb813750',
           'c63295b4ba684c1741bd07666e87a058',
           '414dc440ffe977bace8545f2a2e336a4',
           '7d0eb4db3089fe7a927bfb073b5ed6df',
           '65b71cecdfe2d1e302f27e60fa505edc',
           '4f119c0040805456e7bf9d03e44c3f5b',
           'f4bc23ca0fe64509d8e055b5820c8e8c',
           '2a530974b3c6969ea998ff45d3c0690b',
           '27c18b28ecfda28db2987f05757ecf76',
           '4413fab0a6ff0f153fa10080134379c2',
           'e14f3be6b7b1d57f4135160ad0299c6b',
           '35932eba704eda1c014289aa120a8204',
           '12cbaf75397a8907dbd9a6a4f41043f5',
           '9011c6f71e0c8086f3ac4eb0c775b383',
           'aaacba5eb2b16d4e075071fa8490c9f7',
           'e284db3325de9c447468394942ca6335',
           'ebf195ff8edce298a47f1794f844bdfa',
           'ff3e1e50a325360b9269f6998e748020',
           'cdc45516325c72b0053d57fad4518e07',
           '7eca06e57b613c3a300a103feca4111f',
           '96b2e0218359f690c13f4d8af7adcf7b',
           '36ceac3dde19e4c718e8120781446aa8',
           'a137ebb1af3e658a06b443fe5113a21c',
           '7c8745885b8da399b80fcc8a2ac38396',
           '8dd3bdd0a9d7a58de25fca29343fdd75',
           'd52e2ba9a6ecd5c8e85bd84368a35674',
           'f93c7c1f6a2559124c9c4d4e7be0b7c4',
           '84adf07fce2f1a89baee36c78e4f80a3',
           '457ed122d6e3835e1a0852bda6635546',
           '50a2e7844773d387f487d778477f010e',
           '95aaf77378db0d069533c125a1305bd7',
           '9e90038a0de82ff55e2cb694aa099948',
           'bc32ce8be41ef50b5b21f7a864d7a1f3',
           '743ae58bccee046569ead7fce3f072b3',
           'ab7a341014070f1f1ce78c906eb334a2',
           '529e1eec4f56f10bb953b5940f8055b4',
           'a1d38c8f8159b29f557e2a995d902854',
           '4a305307ab4ae8b94d2565b0a4746c78',
           '90e7fed74be6ed77c52113a8ae7b53c5',
           '8ca30dbfc513da3bd720758cf35d784b',
           '3415b1d6ddabeefddaf1e3d14c6e278b',
           '7bb1fe194df4c137e5497c6e38dfb7b4',
           '382c5142b30ea90c0ee14236c4ad10c0',
           'f41e56a24b441f1c4db28c5006586cb4',
           '86ead864b441a86c9bb58e12d08067f7',
           '83e839f7947c2355031e62f12294f126',
           '36a6e47ce8add52e1d7036dd7e5d3aa0',
           'f218718828a87bd48867ea4c6b903e5f',
           'ec5151af3a49e6cf0bb926722e93dabf',
           'ae31c48a2fbf414cc65b5cab82c9d6c3',
           'df7249830a55b06000aef19fc2fc9ca3',
           'b10be46023b828750a2bf91dca6240c7',
           'c2cb6b55271b8abc3cbbc01214f997fe',
           '69477912114615f1e2b1b07c4ad8f876',
           'ea5fe6905e8e895f1f71012786c253bc',
           'fc59fedd3343a9e7dd577124f992c099',
           '5f19bc9e66fcbef5f5001bfb25d923db',
           '0a62dfed732b7072464d0c195cdf207f',
           '41dbd56b3f2f4ac900aae1583bd9f338',
           '60cd63ea9d892b060d79ef95cdb7416a',
           'ee977692b8245c4f300815d455b9cc47',
           '130d4b65205c5fa79e1bf92612b28235',
           '7fc163c04fdcc4bc20420f94f6723392',
           '102af0664473885f070e7e17ebd737ac',
           '0b6cbac838dfe7f47ea1bd0df00ec282']
```

From here, it is trivial to brute force the hash out one character at a time.

For each "target" hash, we take the last 15 characters of what we have found so far (which is the first 15 characters used to produce that hash), and brute force the last character out, adding it to what we've found.

```py
found = "AAAAAAAAAAAAAAA"
while len(found) - 15 < len(targets):
    print("found:",found)
    isFound = False
    for i in range(255):
        test = found[-15:] + chr(i)
        if targets[len(found)-15][:10] == hash(test.encode("utf-8"))[:10]:
            print("chr:",chr(i))
            found += chr(i)
            isFound = True
    if not isFound:
        print("oh no something wrong")
        break
```

Finally, the flag is acquired.

```
found: AAAAAAAAAAAAAAAgreyhats{B3lanJa_m3_Burg3R_1f_y0u_3njoyed_7he_Ch@ll3n93}
```
