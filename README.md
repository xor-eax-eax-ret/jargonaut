# jargonaut ![pep-8](https://github.com/xor-eax-eax-ret/jargonaut/actions/workflows/pep8.yml/badge.svg)
`jargonaut` is an obfuscator for protecting Python3 code with a few cool features. Most of the techniques I have implemented or plan on implementing are ripped from these excellent [University of Arizona lecture slides](https://www2.cs.arizona.edu/~collberg/Teaching/553/2011/Resources/obfuscation.pdf). 

There aren't many Python obfuscators on GitHub that:
- actually produce functional code when some of Python's more complex features are used
- aren't just a combination of variable renaming, Base64 encoding and `marshal`/`eval` spam
- aren't abandoned / deprecated 

This is probably because more advanced obfuscation techniques (especially ones that touch control flow) are pretty difficult to implement for a dynamically typed language that was built around readibility and simplicity! `jargonaut` aims to fill this gap - check out the Upcoming Features section for planned additions.

Note that this is a proof-of-concept and a work in progress. You should not be using this for anything serious - not only is `jargonaut` probably going to introduce bugs, but deobfuscation will likely be trivial until more features are implemented. 

## Features
- Basic variable, function and parameter renaming (more coming soon)
- Obfuscation of function return values with runtime bytecode patching (work in progress)
- Multiple string obfuscation methods with lambda expressions and others 
- String matching evasion with [Unicode identifier variants](https://blog.phylum.io/malicious-actors-use-unicode-support-in-python-to-evade-detection)
- Obfuscation of integer literals and expressions with [linear mixed boolean arithmetic expressions](https://link.springer.com/chapter/10.1007/978-3-540-77535-5_5)

## Planned improvements
### Upcoming features 
- ~Comment removal~
- Type hint removal
- Renaming class methods and attributes with type inferencing
- Opaque predicates/expressions, with and without interdependence
- String obfuscation using Mealy machines
- Packing 
- Bogus control flow 
- Selective virtualization with custom instruction set for functions 
- Dead code/parameter insertion 
- Control flow flattening (chenxification)
- C function conversion a la [pyarmor](https://github.com/dashingsoft/pyarmor)
- Variable splitting/merging
- Function merging 
### Quality of life
- Logging / debugging
- Obfuscation of entire modules, not just single files 
- Documentation 
- Better performance:
    - I'm not using LibCST to its full extent due to lack of knowledge/skill, and I know for a fact the way I perform transformations is suboptimal 
    - I know using Z3 for linear algebra is probably kind of weird and inefficient. I just couldn't figure out how to do it with `numpy` or `scipy` - if you can figure out a better way, please submit a PR! 

## Usage
```main.py source.py obfuscated.py```
`jargonaut` uses Instagram's [LibCST](https://github.com/Instagram/LibCST) for source code transformations. A transformation is a single operation on the source code's CST, like replacing string literals with obfuscated expressions, or removing comments.

You can configure which transformations are applied and their order of application in `main.py`

## Requirements 
- z3-solver
- numpy
- libcst

## References
- https://blog.phylum.io/malicious-actors-use-unicode-support-in-python-to-evade-detection
- https://peps.python.org/pep-0672/#normalizing-identifiers
- https://peps.python.org/pep-3131/
- https://unicode.org/reports/tr15/
- https://docs.python.org/3/library/ast.html
- https://link.springer.com/chapter/10.1007/978-3-540-77535-5_5
- https://theses.hal.science/tel-01623849/document
- https://bbs.kanxue.com/thread-271574.htm
- https://libcst.readthedocs.io
- https://www2.cs.arizona.edu/~collberg/Teaching/553/2011/Resources/obfuscation.pdf
## Example
### Source code
```
import math
class Val:
    def __init__(self, val):
        self.val = val
        self.stringLiteral = "This is a string literal"

    def addOne(self):
        x = self.val + 1 
        return x 
    
    def addVal(self, val):
        x = self.val + val
        self.val = x

    def doMath(self, x):
        return math.log(x, self.val)
    
    def doFormatString(self):
        return f"This is a format string: {self.val}"

class ValSubclass(Val):
    def __init__(self, val):
        super().__init__(val)
        self.subclass = "I am a subclass"

def outside_func(x):
    x = x ** 2
    return x

def higher_order_func(func, x):
    return func(x)

def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n-1)

if __name__ == "__main__":
    v = Val(2)
    binaryOperation = 1 + 2
    print(v.stringLiteral)
    print(outside_func(v.val))
    print(v.addOne())

    v.addVal(2)
    print(v.val)
    print(v.doFormatString())
    try:
        a = 1 / 0
    except ZeroDivisionError:
        print("This is an exception")
    print(higher_order_func(lambda x: x**2, 3))
    x = ValSubclass(2)
    print(x.subclass)
    print(factorial(10))
```
### Obfuscated code
```
# -*- coding: utf-8 -*
import inspect
from ctypes import memmove
import math

class iiIiliIIIiiI:
    def __init__(self, liIllIlIillI):
        self.val = liIllIlIillI
        self.stringLiteral = (lambda iIIIIiIIiIll, lliIliIilIii, iiIiiiiIiiII, liIiIIiillIl, IIIIilIIIIii, iililllliiIi, llliilililIl, iiilllIiIliI:    (lambda lillIliiilil, illlIillilil, iiliiIiiIiii: lillIliiilil(lillIliiilil, illlIillilil, iiliiIiiIiii))(            lambda lillIliiilil, illlIillilil, iiliiIiiIiii:                bytes([iiliiIiiIiii % illlIillilil]) + lillIliiilil(lillIliiilil, illlIillilil, iiliiIiiIiii // illlIillilil) if iiliiIiiIiii else                (lambda: lillIliiilil).__code__.co_lnotab,            iIIIIiIIiIll << iiilllIiIliI,            (((((iIIIIiIIiIll << liIiIIiillIl) + iIIIIiIIiIll) << liIiIIiillIl) + iiIiiiiIiiII) << ((iiIiiiiIiiII << iililllliiIi) + IIIIilIIIIii)) + (((iiIiiiiIiiII << IIIIilIIIIii) + iiIiiiiIiiII) << ((iiIiiiiIiiII << iililllliiIi) - iiIiiiiIiiII)) + (((iiIiiiiIiiII << iililllliiIi) - llliilililIl) << ((((iiIiiiiIiiII << lliIliIilIii) - iIIIIiIIiIll) << liIiIIiillIl) + iIIIIiIIiIll)) + (((((iiIiiiiIiiII << lliIliIilIii) + iIIIIiIIiIll) << liIiIIiillIl) - IIIIilIIIIii) << ((IIIIilIIIIii << IIIIilIIIIii) + llliilililIl)) - (((iiIiiiiIiiII << iililllliiIi) - llliilililIl) << ((IIIIilIIIIii << IIIIilIIIIii) - (iIIIIiIIiIll << lliIliIilIii))) - (((((iiIiiiiIiiII << lliIliIilIii) + iIIIIiIIiIll) << iiIiiiiIiiII) + iIIIIiIIiIll) << (((((iIIIIiIIiIll << iiIiiiiIiiII) + iIIIIiIIiIll)) << liIiIIiillIl) + (iIIIIiIIiIll << lliIliIilIii))) - (((iIIIIiIIiIll << IIIIilIIIIii) - iIIIIiIIiIll) << (((((iIIIIiIIiIll << iiIiiiiIiiII) + iIIIIiIIiIll)) << liIiIIiillIl) - iiIiiiiIiiII)) + (((((iiIiiiiIiiII << lliIliIilIii) + iIIIIiIIiIll) << liIiIIiillIl) - iIIIIiIIiIll) << ((iIIIIiIIiIll << llliilililIl) - iIIIIiIIiIll)) - ((((((iIIIIiIIiIll << iiIiiiiIiiII) + iIIIIiIIiIll)) << liIiIIiillIl) - iiIiiiiIiiII) << ((((iIIIIiIIiIll << liIiIIiillIl) - iIIIIiIIiIll) << iiIiiiiIiiII) - iiIiiiiIiiII)) + (((((IIIIilIIIIii << lliIliIilIii) - iIIIIiIIiIll) << iiIiiiiIiiII) - iIIIIiIIiIll) << ((llliilililIl << liIiIIiillIl) - (iIIIIiIIiIll << lliIliIilIii))) + (((IIIIilIIIIii << IIIIilIIIIii) - iiIiiiiIiiII) << ((iiIiiiiIiiII << IIIIilIIIIii) + (iIIIIiIIiIll << iIIIIiIIiIll))) + (((llliilililIl << liIiIIiillIl) + iiIiiiiIiiII) << ((((iiIiiiiIiiII << lliIliIilIii) - iIIIIiIIiIll) << iiIiiiiIiiII))) + (((iIIIIiIIiIll << iililllliiIi) + iIIIIiIIiIll) << ((IIIIilIIIIii << liIiIIiillIl) - iIIIIiIIiIll)) - (((((iIIIIiIIiIll << liIiIIiillIl) - iIIIIiIIiIll) << iiIiiiiIiiII) + iiIiiiiIiiII) << (((((iIIIIiIIiIll << iiIiiiiIiiII) + iIIIIiIIiIll)) << iiIiiiiIiiII) - (iIIIIiIIiIll << iIIIIiIIiIll))) - (((iIIIIiIIiIll << iililllliiIi) - iIIIIiIIiIll) << ((iIIIIiIIiIll << iililllliiIi) - iIIIIiIIiIll)) - (((((iiIiiiiIiiII << lliIliIilIii) + iIIIIiIIiIll) << iiIiiiiIiiII) - iiIiiiiIiiII) << ((((iiIiiiiIiiII << lliIliIilIii) + iIIIIiIIiIll) << lliIliIilIii) + iIIIIiIIiIll)) + ((((((iIIIIiIIiIll << iiIiiiiIiiII) + iIIIIiIIiIll)) << iiIiiiiIiiII) + iIIIIiIIiIll) << ((((iiIiiiiIiiII << lliIliIilIii) - iIIIIiIIiIll) << lliIliIilIii) + iIIIIiIIiIll)) + (((llliilililIl << lliIliIilIii) + iIIIIiIIiIll) << ((iIIIIiIIiIll << IIIIilIIIIii) + (iIIIIiIIiIll << iIIIIiIIiIll))) - (((IIIIilIIIIii << lliIliIilIii) - iIIIIiIIiIll) << ((llliilililIl << lliIliIilIii) - iIIIIiIIiIll)) + (((((iiIiiiiIiiII << lliIliIilIii) - iIIIIiIIiIll) << lliIliIilIii) + iIIIIiIIiIll) << ((IIIIilIIIIii << lliIliIilIii) - iIIIIiIIiIll)) + (((IIIIilIIIIii << lliIliIilIii) + iIIIIiIIiIll) << ((IIIIilIIIIii << iIIIIiIIiIll))) + (((iIIIIiIIiIll << liIiIIiillIl) + iIIIIiIIiIll) << iIIIIiIIiIll)        )    )(        *(lambda iIIIIiIIiIll, lliIliIilIii, iiIiiiiIiiII: iIIIIiIIiIll(iIIIIiIIiIll, lliIliIilIii, iiIiiiiIiiII))(            (lambda iIIIIiIIiIll, lliIliIilIii, iiIiiiiIiiII:                [lliIliIilIii(iiIiiiiIiiII[(lambda: iIIIIiIIiIll).__code__.co_nlocals])] +                iIIIIiIIiIll(iIIIIiIIiIll, lliIliIilIii, iiIiiiiIiiII[(lambda lillIliiilil: lillIliiilil).__code__.co_nlocals:]) if iiIiiiiIiiII else []            ),            lambda iIIIIiIIiIll: iIIIIiIIiIll.__code__.co_argcount,            (                lambda iIIIIiIIiIll: iIIIIiIIiIll,                lambda iIIIIiIIiIll, lliIliIilIii: iIIIIiIIiIll,                lambda iIIIIiIIiIll, lliIliIilIii, iiIiiiiIiiII: iIIIIiIIiIll,                lambda iIIIIiIIiIll, lliIliIilIii, iiIiiiiIiiII, liIiIIiillIl: iIIIIiIIiIll,                lambda iIIIIiIIiIll, lliIliIilIii, iiIiiiiIiiII, liIiIIiillIl, IIIIilIIIIii: iIIIIiIIiIll,                lambda iIIIIiIIiIll, lliIliIilIii, iiIiiiiIiiII, liIiIIiillIl, IIIIilIIIIii, iililllliiIi: iIIIIiIIiIll,                lambda iIIIIiIIiIll, lliIliIilIii, iiIiiiiIiiII, liIiIIiillIl, IIIIilIIIIii, iililllliiIi, llliilililIl: iIIIIiIIiIll,                lambda iIIIIiIIiIll, lliIliIilIii, iiIiiiiIiiII, liIiIIiillIl, IIIIilIIIIii, iililllliiIi, llliilililIl, iiilllIiIliI: iIIIIiIIiIll            )        )    ).decode("utf-8")[1:-1]

    def addOne(self):
        lilIIiIlIIii = self.val + (1 * (1984 & 999) + 1 * ~(1984 | 999) + -1 * ~(1984 ^ 999) + -1 * -1) 
        def IliIIiiiIiiI(IIIlIliiIiIi):
            iiillllIllIl = inspect.currentframe().f_back
            IiIiIIiIlIlI = iiillllIllIl.f_code.co_code
            lllIlillliii = iiillllIllIl.f_lasti
            IliiiIiiilii = bytes(IiIiIIiIlIlI)
            memmove(id(IliiiIiiilii)+(-1 * (1984 & 31415) + 1 * (1984 & ~31415) + -2 * 1984 + -4 * (~1984 & 31415) + 4 * 31415 + -1 * ~(1984 ^ 31415) + 1 * ~31415 + -32 * -1) + lllIlillliii + (1 * 420 + 2 * 999 + -2 * (420 | 999) + -1 * ~(420 ^ 999) + 1 * ~999 + -2 * -1), b"\x53\x00", (-1 * (31415 & 1337) + -1 * (31415 & ~1337) + 1 * 31415 + -2 * -1))
            return IIIlIliiIiIi
        IliIIiiiIiiI(lilIIiIlIIii)
        return (lambda IIlIllIIiiii, lIIIIlllIilI, IliIllllliil, IillIIiIIiII, iillIlIiIliI, llillIlIlili, IiIllIliiIIl, llIIiiiiilii:    (lambda iiiIlllillll, IllIiiiliiii, ilIlliIlilIl: iiiIlllillll(iiiIlllillll, IllIiiiliiii, ilIlliIlilIl))(            lambda iiiIlllillll, IllIiiiliiii, ilIlliIlilIl:                bytes([ilIlliIlilIl % IllIiiiliiii]) + iiiIlllillll(iiiIlllillll, IllIiiiliiii, ilIlliIlilIl // IllIiiiliiii) if ilIlliIlilIl else                (lambda: iiiIlllillll).__code__.co_lnotab,            IIlIllIIiiii << llIIiiiiilii,            (((((IIlIllIIiiii << IillIIiIIiII) + IIlIllIIiiii) << IliIllllliil) + IIlIllIIiiii) << ((iillIlIiIliI << IillIIiIIiII) - (IIlIllIIiiii << IIlIllIIiiii))) + (((IliIllllliil << IillIIiIIiII) + IIlIllIIiiii) << (((((IIlIllIIiiii << IliIllllliil) + IIlIllIIiiii)) << IliIllllliil) - (IIlIllIIiiii << IIlIllIIiiii))) + (((((IIlIllIIiiii << IillIIiIIiII) - IIlIllIIiiii) << lIIIIlllIilI) + IIlIllIIiiii) << ((IIlIllIIiiii << llillIlIlili) - (IIlIllIIiiii << IIlIllIIiiii))) + (((IliIllllliil << IillIIiIIiII) + IIlIllIIiiii) << ((IiIllIliiIIl << IliIllllliil) - (IIlIllIIiiii << IIlIllIIiiii))) + (((((IIlIllIIiiii << IillIIiIIiII) - IIlIllIIiiii) << lIIIIlllIilI) + IIlIllIIiiii) << ((IliIllllliil << IillIIiIIiII) - (IIlIllIIiiii << IIlIllIIiiii))) + (((IliIllllliil << IillIIiIIiII) + IIlIllIIiiii) << ((iillIlIiIliI << IliIllllliil) - (IIlIllIIiiii << IIlIllIIiiii))) + (((((IIlIllIIiiii << IillIIiIIiII) - IIlIllIIiiii) << lIIIIlllIilI) + IIlIllIIiiii) << ((((IIlIllIIiiii << IillIIiIIiII) - IIlIllIIiiii) << IIlIllIIiiii))) + (((IliIllllliil << IillIIiIIiII) + IIlIllIIiiii) << ((((IliIllllliil << lIIIIlllIilI) - IIlIllIIiiii) << IIlIllIIiiii))) + (((IIlIllIIiiii << iillIlIiIliI) - IIlIllIIiiii) << ((IIlIllIIiiii << IillIIiIIiII) - IIlIllIIiiii)) - ((((IliIllllliil << lIIIIlllIilI) + IIlIllIIiiii)) << ((iillIlIiIliI << IIlIllIIiiii))) + (((IIlIllIIiiii << IillIIiIIiII) + IIlIllIIiiii) << IIlIllIIiiii)        )    )(        *(lambda IIlIllIIiiii, lIIIIlllIilI, IliIllllliil: IIlIllIIiiii(IIlIllIIiiii, lIIIIlllIilI, IliIllllliil))(            (lambda IIlIllIIiiii, lIIIIlllIilI, IliIllllliil:                [lIIIIlllIilI(IliIllllliil[(lambda: IIlIllIIiiii).__code__.co_nlocals])] +                IIlIllIIiiii(IIlIllIIiiii, lIIIIlllIilI, IliIllllliil[(lambda iiiIlllillll: iiiIlllillll).__code__.co_nlocals:]) if IliIllllliil else []            ),            lambda IIlIllIIiiii: IIlIllIIiiii.__code__.co_argcount,            (                lambda IIlIllIIiiii: IIlIllIIiiii,                lambda IIlIllIIiiii, lIIIIlllIilI: IIlIllIIiiii,                lambda IIlIllIIiiii, lIIIIlllIilI, IliIllllliil: IIlIllIIiiii,                lambda IIlIllIIiiii, lIIIIlllIilI, IliIllllliil, IillIIiIIiII: IIlIllIIiiii,                lambda IIlIllIIiiii, lIIIIlllIilI, IliIllllliil, IillIIiIIiII, iillIlIiIliI: IIlIllIIiiii,                lambda IIlIllIIiiii, lIIIIlllIilI, IliIllllliil, IillIIiIIiII, iillIlIiIliI, llillIlIlili: IIlIllIIiiii,                lambda IIlIllIIiiii, lIIIIlllIilI, IliIllllliil, IillIIiIIiII, iillIlIiIliI, llillIlIlili, IiIllIliiIIl: IIlIllIIiiii,                lambda IIlIllIIiiii, lIIIIlllIilI, IliIllllliil, IillIIiIIiII, iillIlIiIliI, llillIlIlili, IiIllIliiIIl, llIIiiiiilii: IIlIllIIiiii            )        )    ).decode("utf-8")[1:-1]
    
    def addVal(self, IIililIiIlll):
        iiliiiIiiiIi = self.val + IIililIiIlll
        self.val = iiliiiIiiiIi

    def doMath(self, iiIIllilliIi):
        def liIlliIilIlI(iIlIliIliiII):
            iliIlilIlIlI = inspect.currentframe().f_back
            iliIililiIII = iliIlilIlIlI.f_code.co_code
            IIIliIllIlli = iliIlilIlIlI.f_lasti
            IIIliiIlliil = bytes(iliIililiIII)
            memmove(id(IIIliiIlliil)+(-2 * (747 & 1984) + -1 * (747 & ~1984) + -2 * 747 + -4 * (~747 & 1984) + 1 * 1984 + 3 * (747 | 1984) + -32 * -1) + IIIliIllIlli + (-2 * (31415 & 999) + 1 * (31415 & ~999) + 2 * 31415 + 4 * 999 + -1 * (31415 ^ 999) + -3 * (31415 | 999) + -1 * ~(31415 ^ 999) + 1 * ~999 + -2 * -1), b"\x53\x00", (2 * (747 & 31415) + -1 * (747 & ~31415) + -1 * 31415 + 2 * (747 ^ 31415) + -1 * (747 | 31415) + -2 * -1))
            return iIlIliIliiII
        liIlliIilIlI(math.log(iiIIllilliIi, self.val))
        return (lambda iilIIiiIiIiI, IIlIliIiliIl, liIiiIilIlli, lllIlIiiiiIi, llIilIlIIilI, iilIllIlIlil, ililIIliIiIl, IlIlIiilllii:    (lambda liiliIIiilIi, llIIIliIllli, lIilIlIiIliI: liiliIIiilIi(liiliIIiilIi, llIIIliIllli, lIilIlIiIliI))(            lambda liiliIIiilIi, llIIIliIllli, lIilIlIiIliI:                bytes([lIilIlIiIliI % llIIIliIllli]) + liiliIIiilIi(liiliIIiilIi, llIIIliIllli, lIilIlIiIliI // llIIIliIllli) if lIilIlIiIliI else                (lambda: liiliIIiilIi).__code__.co_lnotab,            iilIIiiIiIiI << IlIlIiilllii,            (((((iilIIiiIiIiI << lllIlIiiiiIi) + iilIIiiIiIiI) << liIiiIilIlli) + iilIIiiIiIiI) << ((llIilIlIIilI << lllIlIiiiiIi) - (iilIIiiIiIiI << iilIIiiIiIiI))) + (((liIiiIilIlli << lllIlIiiiiIi) + iilIIiiIiIiI) << (((((iilIIiiIiIiI << liIiiIilIlli) + iilIIiiIiIiI)) << liIiiIilIlli) - (iilIIiiIiIiI << iilIIiiIiIiI))) + (((((iilIIiiIiIiI << lllIlIiiiiIi) - iilIIiiIiIiI) << IIlIliIiliIl) + iilIIiiIiIiI) << ((iilIIiiIiIiI << iilIllIlIlil) - (iilIIiiIiIiI << iilIIiiIiIiI))) + (((liIiiIilIlli << lllIlIiiiiIi) + iilIIiiIiIiI) << ((ililIIliIiIl << liIiiIilIlli) - (iilIIiiIiIiI << iilIIiiIiIiI))) + (((((iilIIiiIiIiI << lllIlIiiiiIi) - iilIIiiIiIiI) << IIlIliIiliIl) + iilIIiiIiIiI) << ((liIiiIilIlli << lllIlIiiiiIi) - (iilIIiiIiIiI << iilIIiiIiIiI))) + (((liIiiIilIlli << lllIlIiiiiIi) + iilIIiiIiIiI) << ((llIilIlIIilI << liIiiIilIlli) - (iilIIiiIiIiI << iilIIiiIiIiI))) + (((((iilIIiiIiIiI << lllIlIiiiiIi) - iilIIiiIiIiI) << IIlIliIiliIl) + iilIIiiIiIiI) << ((((iilIIiiIiIiI << lllIlIiiiiIi) - iilIIiiIiIiI) << iilIIiiIiIiI))) + (((liIiiIilIlli << lllIlIiiiiIi) + iilIIiiIiIiI) << ((((liIiiIilIlli << IIlIliIiliIl) - iilIIiiIiIiI) << iilIIiiIiIiI))) + (((iilIIiiIiIiI << llIilIlIIilI) - iilIIiiIiIiI) << ((iilIIiiIiIiI << lllIlIiiiiIi) - iilIIiiIiIiI)) - ((((liIiiIilIlli << IIlIliIiliIl) + iilIIiiIiIiI)) << ((llIilIlIIilI << iilIIiiIiIiI))) + (((iilIIiiIiIiI << lllIlIiiiiIi) + iilIIiiIiIiI) << iilIIiiIiIiI)        )    )(        *(lambda iilIIiiIiIiI, IIlIliIiliIl, liIiiIilIlli: iilIIiiIiIiI(iilIIiiIiIiI, IIlIliIiliIl, liIiiIilIlli))(            (lambda iilIIiiIiIiI, IIlIliIiliIl, liIiiIilIlli:                [IIlIliIiliIl(liIiiIilIlli[(lambda: iilIIiiIiIiI).__code__.co_nlocals])] +                iilIIiiIiIiI(iilIIiiIiIiI, IIlIliIiliIl, liIiiIilIlli[(lambda liiliIIiilIi: liiliIIiilIi).__code__.co_nlocals:]) if liIiiIilIlli else []            ),            lambda iilIIiiIiIiI: iilIIiiIiIiI.__code__.co_argcount,            (                lambda iilIIiiIiIiI: iilIIiiIiIiI,                lambda iilIIiiIiIiI, IIlIliIiliIl: iilIIiiIiIiI,                lambda iilIIiiIiIiI, IIlIliIiliIl, liIiiIilIlli: iilIIiiIiIiI,                lambda iilIIiiIiIiI, IIlIliIiliIl, liIiiIilIlli, lllIlIiiiiIi: iilIIiiIiIiI,                lambda iilIIiiIiIiI, IIlIliIiliIl, liIiiIilIlli, lllIlIiiiiIi, llIilIlIIilI: iilIIiiIiIiI,                lambda iilIIiiIiIiI, IIlIliIiliIl, liIiiIilIlli, lllIlIiiiiIi, llIilIlIIilI, iilIllIlIlil: iilIIiiIiIiI,                lambda iilIIiiIiIiI, IIlIliIiliIl, liIiiIilIlli, lllIlIiiiiIi, llIilIlIIilI, iilIllIlIlil, ililIIliIiIl: iilIIiiIiIiI,                lambda iilIIiiIiIiI, IIlIliIiliIl, liIiiIilIlli, lllIlIiiiiIi, llIilIlIIilI, iilIllIlIlil, ililIIliIiIl, IlIlIiilllii: iilIIiiIiIiI            )        )    ).decode("utf-8")[1:-1]
    
    def doFormatString(self):
        def iIIIlIIIiiiI(iIIIlIliiill):
            iiIlIilIIIlI = inspect.currentframe().f_back
            llIIIiIiiIiI = iiIlIilIIIlI.f_code.co_code
            iIIlIiIIIiiI = iiIlIilIIIlI.f_lasti
            liIililliill = bytes(llIIIiIiiIiI)
            memmove(id(liIililliill)+(-2 * (1984 & ~31415) + 5 * 1984 + 7 * (~1984 & 31415) + -6 * 31415 + -1 * (1984 | 31415) + 2 * ~(1984 ^ 31415) + -2 * ~31415 + -32 * -1) + iIIlIiIIIiiI + (4 * (1984 & ~420) + -2 * 1984 + -2 * (1984 ^ 420) + 2 * (1984 | 420) + 2 * ~(1984 | 420) + -2 * ~420 + -2 * -1), b"\x53\x00", (-2 * (1984 & ~747) + 1 * 1984 + -1 * ~(1984 ^ 747) + 1 * ~747 + -2 * -1))
            return iIIIlIliiill
        iIIIlIIIiiiI(f"This is a format string: {self.val}")
        return (lambda lIilIiiiliiI, iliIiIllilII, ilIiiiIIlliI, IIiIilliilIl, lliiIiIIilli, iIiiiilIliii, IilIIliiIIIi, iliIIliIllii:    (lambda llIlIilliliI, iiilliilIlii, iIiIllllIIII: llIlIilliliI(llIlIilliliI, iiilliilIlii, iIiIllllIIII))(            lambda llIlIilliliI, iiilliilIlii, iIiIllllIIII:                bytes([iIiIllllIIII % iiilliilIlii]) + llIlIilliliI(llIlIilliliI, iiilliilIlii, iIiIllllIIII // iiilliilIlii) if iIiIllllIIII else                (lambda: llIlIilliliI).__code__.co_lnotab,            lIilIiiiliiI << iliIIliIllii,            (((((lIilIiiiliiI << IIiIilliilIl) + lIilIiiiliiI) << ilIiiiIIlliI) + lIilIiiiliiI) << ((lliiIiIIilli << IIiIilliilIl) - (lIilIiiiliiI << lIilIiiiliiI))) + (((ilIiiiIIlliI << IIiIilliilIl) + lIilIiiiliiI) << (((((lIilIiiiliiI << ilIiiiIIlliI) + lIilIiiiliiI)) << ilIiiiIIlliI) - (lIilIiiiliiI << lIilIiiiliiI))) + (((((lIilIiiiliiI << IIiIilliilIl) - lIilIiiiliiI) << iliIiIllilII) + lIilIiiiliiI) << ((lIilIiiiliiI << iIiiiilIliii) - (lIilIiiiliiI << lIilIiiiliiI))) + (((ilIiiiIIlliI << IIiIilliilIl) + lIilIiiiliiI) << ((IilIIliiIIIi << ilIiiiIIlliI) - (lIilIiiiliiI << lIilIiiiliiI))) + (((((lIilIiiiliiI << IIiIilliilIl) - lIilIiiiliiI) << iliIiIllilII) + lIilIiiiliiI) << ((ilIiiiIIlliI << IIiIilliilIl) - (lIilIiiiliiI << lIilIiiiliiI))) + (((ilIiiiIIlliI << IIiIilliilIl) + lIilIiiiliiI) << ((lliiIiIIilli << ilIiiiIIlliI) - (lIilIiiiliiI << lIilIiiiliiI))) + (((((lIilIiiiliiI << IIiIilliilIl) - lIilIiiiliiI) << iliIiIllilII) + lIilIiiiliiI) << ((((lIilIiiiliiI << IIiIilliilIl) - lIilIiiiliiI) << lIilIiiiliiI))) + (((ilIiiiIIlliI << IIiIilliilIl) + lIilIiiiliiI) << ((((ilIiiiIIlliI << iliIiIllilII) - lIilIiiiliiI) << lIilIiiiliiI))) + (((lIilIiiiliiI << lliiIiIIilli) - lIilIiiiliiI) << ((lIilIiiiliiI << IIiIilliilIl) - lIilIiiiliiI)) - ((((ilIiiiIIlliI << iliIiIllilII) + lIilIiiiliiI)) << ((lliiIiIIilli << lIilIiiiliiI))) + (((lIilIiiiliiI << IIiIilliilIl) + lIilIiiiliiI) << lIilIiiiliiI)        )    )(        *(lambda lIilIiiiliiI, iliIiIllilII, ilIiiiIIlliI: lIilIiiiliiI(lIilIiiiliiI, iliIiIllilII, ilIiiiIIlliI))(            (lambda lIilIiiiliiI, iliIiIllilII, ilIiiiIIlliI:                [iliIiIllilII(ilIiiiIIlliI[(lambda: lIilIiiiliiI).__code__.co_nlocals])] +                lIilIiiiliiI(lIilIiiiliiI, iliIiIllilII, ilIiiiIIlliI[(lambda llIlIilliliI: llIlIilliliI).__code__.co_nlocals:]) if ilIiiiIIlliI else []            ),            lambda lIilIiiiliiI: lIilIiiiliiI.__code__.co_argcount,            (                lambda lIilIiiiliiI: lIilIiiiliiI,                lambda lIilIiiiliiI, iliIiIllilII: lIilIiiiliiI,                lambda lIilIiiiliiI, iliIiIllilII, ilIiiiIIlliI: lIilIiiiliiI,                lambda lIilIiiiliiI, iliIiIllilII, ilIiiiIIlliI, IIiIilliilIl: lIilIiiiliiI,                lambda lIilIiiiliiI, iliIiIllilII, ilIiiiIIlliI, IIiIilliilIl, lliiIiIIilli: lIilIiiiliiI,                lambda lIilIiiiliiI, iliIiIllilII, ilIiiiIIlliI, IIiIilliilIl, lliiIiIIilli, iIiiiilIliii: lIilIiiiliiI,                lambda lIilIiiiliiI, iliIiIllilII, ilIiiiIIlliI, IIiIilliilIl, lliiIiIIilli, iIiiiilIliii, IilIIliiIIIi: lIilIiiiliiI,                lambda lIilIiiiliiI, iliIiIllilII, ilIiiiIIlliI, IIiIilliilIl, lliiIiIIilli, iIiiiilIliii, IilIIliiIIIi, iliIIliIllii: lIilIiiiliiI            )        )    ).decode("utf-8")[1:-1]

class iIllIIIiIliI(iiIiliIIIiiI):
    def __init__(self, IiiiIIlIilIi):
        super().__init__(IiiiIIlIilIi)
        self.subclass = (lambda iiIIIlillIll, iiliIIllIiii, lilIilllliiI, IllllilIIiiI, liiiIiilIIII, IllillIiIiil, iiiiIlIiIill, iiIIIIiiIIIi:    (lambda lIiilIllllll, llIliiliIiii, IliIiIIllliI: lIiilIllllll(lIiilIllllll, llIliiliIiii, IliIiIIllliI))(            lambda lIiilIllllll, llIliiliIiii, IliIiIIllliI:                bytes([IliIiIIllliI % llIliiliIiii]) + lIiilIllllll(lIiilIllllll, llIliiliIiii, IliIiIIllliI // llIliiliIiii) if IliIiIIllliI else                (lambda: lIiilIllllll).__code__.co_lnotab,            iiIIIlillIll << iiIIIIiiIIIi,            (((((iiIIIlillIll << IllllilIIiiI) + iiIIIlillIll) << iiliIIllIiii) + iiIIIlillIll) << ((iiIIIlillIll << iiiiIlIiIill) - iiIIIlillIll)) - (((((lilIilllliiI << iiliIIllIiii) + iiIIIlillIll) << IllllilIIiiI) - iiiiIlIiIill) << ((iiiiIlIiIill << IllllilIIiiI) + (iiIIIlillIll << iiliIIllIiii))) + (((iiiiIlIiIill << iiliIIllIiii) - iiIIIlillIll) << ((iiiiIlIiIill << IllllilIIiiI) - lilIilllliiI)) + (((((lilIilllliiI << iiliIIllIiii) - iiIIIlillIll) << lilIilllliiI) + lilIilllliiI) << ((lilIilllliiI << liiiIiilIIII) + (iiIIIlillIll << iiIIIlillIll))) + (((lilIilllliiI << liiiIiilIIII) + lilIilllliiI) << ((((lilIilllliiI << iiliIIllIiii) - iiIIIlillIll) << lilIilllliiI))) + (((lilIilllliiI << IllllilIIiiI) + iiIIIlillIll) << ((liiiIiilIIII << IllllilIIiiI) + iiIIIlillIll)) + (((((iiIIIlillIll << IllllilIIiiI) - iiIIIlillIll) << lilIilllliiI) - lilIilllliiI) << (((((iiIIIlillIll << lilIilllliiI) + iiIIIlillIll)) << lilIilllliiI))) + (((iiiiIlIiIill << IllllilIIiiI) + lilIilllliiI) << ((iiIIIlillIll << IllillIiIiil))) + (((iiIIIlillIll << IllillIiIiil) + iiIIIlillIll) << ((iiiiIlIiIill << lilIilllliiI) - iiIIIlillIll)) - (((iiIIIlillIll << liiiIiilIIII) - iiIIIlillIll) << ((lilIilllliiI << IllllilIIiiI))) + (((iiIIIlillIll << IllillIiIiil) + iiIIIlillIll) << ((liiiIiilIIII << lilIilllliiI) - iiIIIlillIll)) - ((((((iiIIIlillIll << lilIilllliiI) + iiIIIlillIll)) << iiliIIllIiii) + iiIIIlillIll) << ((iiIIIlillIll << liiiIiilIIII) - iiIIIlillIll)) - (((iiIIIlillIll << liiiIiilIIII) - iiIIIlillIll) << ((lilIilllliiI << lilIilllliiI))) + (iiIIIlillIll << ((liiiIiilIIII << iiliIIllIiii) + iiIIIlillIll)) + ((((((iiIIIlillIll << lilIilllliiI) + iiIIIlillIll)) << iiliIIllIiii) + iiIIIlillIll) << ((((iiIIIlillIll << lilIilllliiI) + iiIIIlillIll)))) - (iiiiIlIiIill << liiiIiilIIII) + (iiIIIlillIll << iiIIIlillIll)        )    )(        *(lambda iiIIIlillIll, iiliIIllIiii, lilIilllliiI: iiIIIlillIll(iiIIIlillIll, iiliIIllIiii, lilIilllliiI))(            (lambda iiIIIlillIll, iiliIIllIiii, lilIilllliiI:                [iiliIIllIiii(lilIilllliiI[(lambda: iiIIIlillIll).__code__.co_nlocals])] +                iiIIIlillIll(iiIIIlillIll, iiliIIllIiii, lilIilllliiI[(lambda lIiilIllllll: lIiilIllllll).__code__.co_nlocals:]) if lilIilllliiI else []            ),            lambda iiIIIlillIll: iiIIIlillIll.__code__.co_argcount,            (                lambda iiIIIlillIll: iiIIIlillIll,                lambda iiIIIlillIll, iiliIIllIiii: iiIIIlillIll,                lambda iiIIIlillIll, iiliIIllIiii, lilIilllliiI: iiIIIlillIll,                lambda iiIIIlillIll, iiliIIllIiii, lilIilllliiI, IllllilIIiiI: iiIIIlillIll,                lambda iiIIIlillIll, iiliIIllIiii, lilIilllliiI, IllllilIIiiI, liiiIiilIIII: iiIIIlillIll,                lambda iiIIIlillIll, iiliIIllIiii, lilIilllliiI, IllllilIIiiI, liiiIiilIIII, IllillIiIiil: iiIIIlillIll,                lambda iiIIIlillIll, iiliIIllIiii, lilIilllliiI, IllllilIIiiI, liiiIiilIIII, IllillIiIiil, iiiiIlIiIill: iiIIIlillIll,                lambda iiIIIlillIll, iiliIIllIiii, lilIilllliiI, IllllilIIiiI, liiiIiilIIII, IllillIiIiil, iiiiIlIiIill, iiIIIIiiIIIi: iiIIIlillIll            )        )    ).decode("utf-8")[1:-1]

def iIIillIillii(IllIliliIiIl):
    IllIliliIiIl = IllIliliIiIl ** (-1 * (420 & 1984) + -1 * (~420 & 1984) + 1 * 1984 + -1 * (420 ^ 1984) + 1 * (420 | 1984) + 1 * ~(420 | 1984) + -1 * ~(420 ^ 1984) + -2 * -1)
    def IlIlIiiiiili(iiiIiIiIIIiI):
        iIliIilIIill = inspect.currentframe().f_back
        llIIililliil = iIliIilIIill.f_code.co_code
        iIIliIlIIlIl = iIliIilIIill.f_lasti
        liIlliiIilIl = bytes(llIIililliil)
        memmove(id(liIlliiIilIl)+(-3 * (747 & 1984) + -2 * (747 & ~1984) + -1 * (~747 & 1984) + -1 * 1984 + -2 * (747 ^ 1984) + 4 * (747 | 1984) + -32 * -1) + iIIliIlIIlIl + (-2 * (1337 & 1984) + 1 * (1337 & ~1984) + 1 * (~1337 & 1984) + -1 * (1337 | 1984) + -3 * ~(1337 | 1984) + 3 * ~(1337 ^ 1984) + -2 * -1), b"\x53\x00", (-1 * (31415 & 1984) + 3 * 31415 + 3 * 1984 + -3 * (31415 | 1984) + 2 * ~(31415 | 1984) + -2 * ~(31415 ^ 1984) + -2 * -1))
        return iiiIiIiIIIiI
    IlIlIiiiiili(IllIliliIiIl)
    return (lambda lIIiiiiIilII, liIIIIIlIlll, IIiiiIlIIllI, liiIllIIilIi, lillIlIiIlii, lilIlIiiIiil, iIlillIlIIil, IlliIIliIIli:    (lambda iIllliiIlIlI, illlIilIIIlI, iilIIiIilIli: iIllliiIlIlI(iIllliiIlIlI, illlIilIIIlI, iilIIiIilIli))(            lambda iIllliiIlIlI, illlIilIIIlI, iilIIiIilIli:                bytes([iilIIiIilIli % illlIilIIIlI]) + iIllliiIlIlI(iIllliiIlIlI, illlIilIIIlI, iilIIiIilIli // illlIilIIIlI) if iilIIiIilIli else                (lambda: iIllliiIlIlI).__code__.co_lnotab,            lIIiiiiIilII << IlliIIliIIli,            (((((lIIiiiiIilII << liiIllIIilIi) + lIIiiiiIilII) << IIiiiIlIIllI) + lIIiiiiIilII) << ((lillIlIiIlii << liiIllIIilIi) - (lIIiiiiIilII << lIIiiiiIilII))) + (((IIiiiIlIIllI << liiIllIIilIi) + lIIiiiiIilII) << (((((lIIiiiiIilII << IIiiiIlIIllI) + lIIiiiiIilII)) << IIiiiIlIIllI) - (lIIiiiiIilII << lIIiiiiIilII))) + (((((lIIiiiiIilII << liiIllIIilIi) - lIIiiiiIilII) << liIIIIIlIlll) + lIIiiiiIilII) << ((lIIiiiiIilII << lilIlIiiIiil) - (lIIiiiiIilII << lIIiiiiIilII))) + (((IIiiiIlIIllI << liiIllIIilIi) + lIIiiiiIilII) << ((iIlillIlIIil << IIiiiIlIIllI) - (lIIiiiiIilII << lIIiiiiIilII))) + (((((lIIiiiiIilII << liiIllIIilIi) - lIIiiiiIilII) << liIIIIIlIlll) + lIIiiiiIilII) << ((IIiiiIlIIllI << liiIllIIilIi) - (lIIiiiiIilII << lIIiiiiIilII))) + (((IIiiiIlIIllI << liiIllIIilIi) + lIIiiiiIilII) << ((lillIlIiIlii << IIiiiIlIIllI) - (lIIiiiiIilII << lIIiiiiIilII))) + (((((lIIiiiiIilII << liiIllIIilIi) - lIIiiiiIilII) << liIIIIIlIlll) + lIIiiiiIilII) << ((((lIIiiiiIilII << liiIllIIilIi) - lIIiiiiIilII) << lIIiiiiIilII))) + (((IIiiiIlIIllI << liiIllIIilIi) + lIIiiiiIilII) << ((((IIiiiIlIIllI << liIIIIIlIlll) - lIIiiiiIilII) << lIIiiiiIilII))) + (((lIIiiiiIilII << lillIlIiIlii) - lIIiiiiIilII) << ((lIIiiiiIilII << liiIllIIilIi) - lIIiiiiIilII)) - ((((IIiiiIlIIllI << liIIIIIlIlll) + lIIiiiiIilII)) << ((lillIlIiIlii << lIIiiiiIilII))) + (((lIIiiiiIilII << liiIllIIilIi) + lIIiiiiIilII) << lIIiiiiIilII)        )    )(        *(lambda lIIiiiiIilII, liIIIIIlIlll, IIiiiIlIIllI: lIIiiiiIilII(lIIiiiiIilII, liIIIIIlIlll, IIiiiIlIIllI))(            (lambda lIIiiiiIilII, liIIIIIlIlll, IIiiiIlIIllI:                [liIIIIIlIlll(IIiiiIlIIllI[(lambda: lIIiiiiIilII).__code__.co_nlocals])] +                lIIiiiiIilII(lIIiiiiIilII, liIIIIIlIlll, IIiiiIlIIllI[(lambda iIllliiIlIlI: iIllliiIlIlI).__code__.co_nlocals:]) if IIiiiIlIIllI else []            ),            lambda lIIiiiiIilII: lIIiiiiIilII.__code__.co_argcount,            (                lambda lIIiiiiIilII: lIIiiiiIilII,                lambda lIIiiiiIilII, liIIIIIlIlll: lIIiiiiIilII,                lambda lIIiiiiIilII, liIIIIIlIlll, IIiiiIlIIllI: lIIiiiiIilII,                lambda lIIiiiiIilII, liIIIIIlIlll, IIiiiIlIIllI, liiIllIIilIi: lIIiiiiIilII,                lambda lIIiiiiIilII, liIIIIIlIlll, IIiiiIlIIllI, liiIllIIilIi, lillIlIiIlii: lIIiiiiIilII,                lambda lIIiiiiIilII, liIIIIIlIlll, IIiiiIlIIllI, liiIllIIilIi, lillIlIiIlii, lilIlIiiIiil: lIIiiiiIilII,                lambda lIIiiiiIilII, liIIIIIlIlll, IIiiiIlIIllI, liiIllIIilIi, lillIlIiIlii, lilIlIiiIiil, iIlillIlIIil: lIIiiiiIilII,                lambda lIIiiiiIilII, liIIIIIlIlll, IIiiiIlIIllI, liiIllIIilIi, lillIlIiIlii, lilIlIiiIiil, iIlillIlIIil, IlliIIliIIli: lIIiiiiIilII            )        )    ).decode("utf-8")[1:-1]

def iIIiIliiillI(IIIIIiIilIII, lliIIlilliil):
    def liiIlIliiIil(IIiiiIIlilIi):
        liiiiilIIIIl = inspect.currentframe().f_back
        liIlIIilliiI = liiiiilIIIIl.f_code.co_code
        IillIIIiIIll = liiiiilIIIIl.f_lasti
        lilIilIiilII = bytes(liIlIIilliiI)
        memmove(id(lilIilIiilII)+(-4 * (1984 & 31415) + -1 * (~1984 & 31415) + -2 * (1984 ^ 31415) + 3 * (1984 | 31415) + 1 * ~(1984 ^ 31415) + -1 * ~31415 + -32 * -1) + IillIIIiIIll + (-1 * (1337 & 747) + -1 * 1337 + 2 * 747 + -2 * (1337 ^ 747) + -3 * ~(1337 | 747) + 3 * ~747 + -2 * -1), b"\x53\x00", (-4 * (747 & 999) + -1 * (747 & ~999) + -1 * 747 + -1 * (~747 & 999) + -1 * 999 + -4 * (747 ^ 999) + 6 * (747 | 999) + -2 * -1))
        return IIiiiIIlilIi
    liiIlIliiIil(IIIIIiIilIII(lliIIlilliil))
    return (lambda IIIliIlIiiiI, lIIiiiIiIIll, IililIIIIill, lliiiIiIiIlI, liilililIIII, llliliIIiIil, iIIIIIiIIIli, iIIiIIiliilI:    (lambda liiilIIiIIii, IlIllIllIlll, llIIiiiiIIlI: liiilIIiIIii(liiilIIiIIii, IlIllIllIlll, llIIiiiiIIlI))(            lambda liiilIIiIIii, IlIllIllIlll, llIIiiiiIIlI:                bytes([llIIiiiiIIlI % IlIllIllIlll]) + liiilIIiIIii(liiilIIiIIii, IlIllIllIlll, llIIiiiiIIlI // IlIllIllIlll) if llIIiiiiIIlI else                (lambda: liiilIIiIIii).__code__.co_lnotab,            IIIliIlIiiiI << iIIiIIiliilI,            (((((IIIliIlIiiiI << lliiiIiIiIlI) + IIIliIlIiiiI) << IililIIIIill) + IIIliIlIiiiI) << ((liilililIIII << lliiiIiIiIlI) - (IIIliIlIiiiI << IIIliIlIiiiI))) + (((IililIIIIill << lliiiIiIiIlI) + IIIliIlIiiiI) << (((((IIIliIlIiiiI << IililIIIIill) + IIIliIlIiiiI)) << IililIIIIill) - (IIIliIlIiiiI << IIIliIlIiiiI))) + (((((IIIliIlIiiiI << lliiiIiIiIlI) - IIIliIlIiiiI) << lIIiiiIiIIll) + IIIliIlIiiiI) << ((IIIliIlIiiiI << llliliIIiIil) - (IIIliIlIiiiI << IIIliIlIiiiI))) + (((IililIIIIill << lliiiIiIiIlI) + IIIliIlIiiiI) << ((iIIIIIiIIIli << IililIIIIill) - (IIIliIlIiiiI << IIIliIlIiiiI))) + (((((IIIliIlIiiiI << lliiiIiIiIlI) - IIIliIlIiiiI) << lIIiiiIiIIll) + IIIliIlIiiiI) << ((IililIIIIill << lliiiIiIiIlI) - (IIIliIlIiiiI << IIIliIlIiiiI))) + (((IililIIIIill << lliiiIiIiIlI) + IIIliIlIiiiI) << ((liilililIIII << IililIIIIill) - (IIIliIlIiiiI << IIIliIlIiiiI))) + (((((IIIliIlIiiiI << lliiiIiIiIlI) - IIIliIlIiiiI) << lIIiiiIiIIll) + IIIliIlIiiiI) << ((((IIIliIlIiiiI << lliiiIiIiIlI) - IIIliIlIiiiI) << IIIliIlIiiiI))) + (((IililIIIIill << lliiiIiIiIlI) + IIIliIlIiiiI) << ((((IililIIIIill << lIIiiiIiIIll) - IIIliIlIiiiI) << IIIliIlIiiiI))) + (((IIIliIlIiiiI << liilililIIII) - IIIliIlIiiiI) << ((IIIliIlIiiiI << lliiiIiIiIlI) - IIIliIlIiiiI)) - ((((IililIIIIill << lIIiiiIiIIll) + IIIliIlIiiiI)) << ((liilililIIII << IIIliIlIiiiI))) + (((IIIliIlIiiiI << lliiiIiIiIlI) + IIIliIlIiiiI) << IIIliIlIiiiI)        )    )(        *(lambda IIIliIlIiiiI, lIIiiiIiIIll, IililIIIIill: IIIliIlIiiiI(IIIliIlIiiiI, lIIiiiIiIIll, IililIIIIill))(            (lambda IIIliIlIiiiI, lIIiiiIiIIll, IililIIIIill:                [lIIiiiIiIIll(IililIIIIill[(lambda: IIIliIlIiiiI).__code__.co_nlocals])] +                IIIliIlIiiiI(IIIliIlIiiiI, lIIiiiIiIIll, IililIIIIill[(lambda liiilIIiIIii: liiilIIiIIii).__code__.co_nlocals:]) if IililIIIIill else []            ),            lambda IIIliIlIiiiI: IIIliIlIiiiI.__code__.co_argcount,            (                lambda IIIliIlIiiiI: IIIliIlIiiiI,                lambda IIIliIlIiiiI, lIIiiiIiIIll: IIIliIlIiiiI,                lambda IIIliIlIiiiI, lIIiiiIiIIll, IililIIIIill: IIIliIlIiiiI,                lambda IIIliIlIiiiI, lIIiiiIiIIll, IililIIIIill, lliiiIiIiIlI: IIIliIlIiiiI,                lambda IIIliIlIiiiI, lIIiiiIiIIll, IililIIIIill, lliiiIiIiIlI, liilililIIII: IIIliIlIiiiI,                lambda IIIliIlIiiiI, lIIiiiIiIIll, IililIIIIill, lliiiIiIiIlI, liilililIIII, llliliIIiIil: IIIliIlIiiiI,                lambda IIIliIlIiiiI, lIIiiiIiIIll, IililIIIIill, lliiiIiIiIlI, liilililIIII, llliliIIiIil, iIIIIIiIIIli: IIIliIlIiiiI,                lambda IIIliIlIiiiI, lIIiiiIiIIll, IililIIIIill, lliiiIiIiIlI, liilililIIII, llliliIIiIil, iIIIIIiIIIli, iIIiIIiliilI: IIIliIlIiiiI            )        )    ).decode("utf-8")[1:-1]

def ilIiIlllIIli(iiiIIlIliiIi):
    if iiiIIlIliiIi == (-2 * (999 & 1984) + -2 * 999 + 1 * (~999 & 1984) + -3 * 1984 + 2 * (999 | 1984) + -5 * ~(999 | 1984) + 5 * ~(999 ^ 1984)):
        return (1 * (999 & 420) + -2 * (999 & ~420) + -3 * 999 + -5 * 420 + -1 * (999 ^ 420) + 6 * (999 | 420) + -1 * ~(999 | 420) + 1 * ~(999 ^ 420) + -1 * -1)
    else:
        return iiiIIlIliiIi * ilIiIlllIIli(iiiIIlIliiIi-(5 * (999 & ~1984) + -2 * 999 + 3 * (~999 & 1984) + -1 * 1984 + -2 * (999 ^ 1984) + -2 * ~(999 | 1984) + 3 * ~(999 ^ 1984) + -1 * ~1984 + -1 * -1))

if __name__ == (lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll, IIliIililllI, lllilIlIiiIi, iiIlIiIiIiIi, IIlIlilIIIII:    (lambda iiiiiiililli, llllIIIllIIl, lIIillIIIili: iiiiiiililli(iiiiiiililli, llllIIIllIIl, lIIillIIIili))(            lambda iiiiiiililli, llllIIIllIIl, lIIillIIIili:                bytes([lIIillIIIili % llllIIIllIIl]) + iiiiiiililli(iiiiiiililli, llllIIIllIIl, lIIillIIIili // llllIIIllIIl) if lIIillIIIili else                (lambda: iiiiiiililli).__code__.co_lnotab,            lllilliiiill << IIlIlilIIIII,            (((((lllilliiiill << IillillIlIll) + lllilliiiill) << iIIllliliiIl) + lllilliiiill) << (((((lllilliiiill << iIIllliliiIl) + lllilliiiill)) << iIIllliliiIl) - (lllilliiiill << lllilliiiill))) + (((lllilliiiill << iiIlIiIiIiIi) - iIIllliliiIl) << ((lllilliiiill << lllilIlIiiIi) - (lllilliiiill << lllilliiiill))) + (((lllilliiiill << lllilIlIiiIi) - lllilliiiill) << ((iiIlIiIiIiIi << iIIllliliiIl) - lllilliiiill)) - ((((((lllilliiiill << iIIllliliiIl) + lllilliiiill)) << IlIiiliIIlii) - lllilliiiill) << ((iIIllliliiIl << IillillIlIll) - lllilliiiill)) - (((((iIIllliliiIl << IlIiiliIIlii) - lllilliiiill) << IlIiiliIIlii) + lllilliiiill) << ((IIliIililllI << iIIllliliiIl) - lllilliiiill)) - (((((lllilliiiill << IillillIlIll) - lllilliiiill) << IlIiiliIIlii) + lllilliiiill) << ((lllilliiiill << IIliIililllI) - lllilliiiill)) - ((((((lllilliiiill << iIIllliliiIl) + lllilliiiill)) << IlIiiliIIlii) + lllilliiiill) << ((iIIllliliiIl << iIIllliliiIl) - lllilliiiill)) - (((lllilliiiill << IIliIililllI) + lllilliiiill) << ((lllilliiiill << IillillIlIll))) + (iIIllliliiIl << (((iIIllliliiIl << IlIiiliIIlii) + lllilliiiill))) - (iiIlIiIiIiIi << IIliIililllI) + (lllilliiiill << lllilliiiill)        )    )(        *(lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl: lllilliiiill(lllilliiiill, IlIiiliIIlii, iIIllliliiIl))(            (lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl:                [IlIiiliIIlii(iIIllliliiIl[(lambda: lllilliiiill).__code__.co_nlocals])] +                lllilliiiill(lllilliiiill, IlIiiliIIlii, iIIllliliiIl[(lambda iiiiiiililli: iiiiiiililli).__code__.co_nlocals:]) if iIIllliliiIl else []            ),            lambda lllilliiiill: lllilliiiill.__code__.co_argcount,            (                lambda lllilliiiill: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll, IIliIililllI: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll, IIliIililllI, lllilIlIiiIi: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll, IIliIililllI, lllilIlIiiIi, iiIlIiIiIiIi: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll, IIliIililllI, lllilIlIiiIi, iiIlIiIiIiIi, IIlIlilIIIII: lllilliiiill            )        )    ).decode("utf-8")[1:-1]:
    IilIlliIIilI = iiIiliIIIiiI((-3 * (31415 & 1984) + 1 * (31415 & ~1984) + 1 * 31415 + -1 * (~31415 & 1984) + 3 * 1984 + -2 * (31415 | 1984) + -1 * ~(31415 | 1984) + 1 * ~(31415 ^ 1984) + -2 * -1))
    IlIIiliiiIIl = (-2 * (999 & ~1337) + -1 * 999 + -2 * (~999 & 1337) + -1 * 1337 + 1 * (999 ^ 1337) + 2 * (999 | 1337) + -1 * -1) + (-1 * (747 & 1984) + -2 * (747 & ~1984) + -1 * 747 + -1 * 1984 + -2 * (747 ^ 1984) + 3 * (747 | 1984) + -2 * ~(747 | 1984) + 2 * ~1984 + -2 * -1)
    print(IilIlliIIilI.stringLiteral)
    print(iIIillIillii(IilIlliIIilI.val))
    print(IilIlliIIilI.addOne())

    IilIlliIIilI.addVal((-1 * (1984 & 747) + -2 * 747 + 1 * (1984 ^ 747) + 1 * (1984 | 747) + 2 * ~(1984 ^ 747) + -2 * ~747 + -2 * -1))
    print(IilIlliIIilI.val)
    print(IilIlliIIilI.doFormatString())
    try:
        iIiliIilIIIl = (-1 * (999 & 747) + -1 * (~999 & 747) + 2 * 747 + -1 * (999 | 747) + -1 * ~(999 | 747) + 1 * ~747 + -1 * -1) / (5 * (999 & ~747) + -3 * 999 + -1 * (~999 & 747) + 2 * 747 + -1 * (999 ^ 747) + 1 * ~(999 ^ 747) + -1 * ~747)
    except ZeroDivisionError:
        print((lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll, IIliIililllI, lllilIlIiiIi, iiIlIiIiIiIi, IIlIlilIIIII:    (lambda iiiiiiililli, llllIIIllIIl, lIIillIIIili: iiiiiiililli(iiiiiiililli, llllIIIllIIl, lIIillIIIili))(            lambda iiiiiiililli, llllIIIllIIl, lIIillIIIili:                bytes([lIIillIIIili % llllIIIllIIl]) + iiiiiiililli(iiiiiiililli, llllIIIllIIl, lIIillIIIili // llllIIIllIIl) if lIIillIIIili else                (lambda: iiiiiiililli).__code__.co_lnotab,            lllilliiiill << IIlIlilIIIII,            (((((lllilliiiill << IillillIlIll) + lllilliiiill) << IillillIlIll) + iIIllliliiIl) << ((IIliIililllI << IIliIililllI) + IIliIililllI)) + (((iiIlIiIiIiIi << IIliIililllI) + iiIlIiIiIiIi) << ((IIliIililllI << IIliIililllI) - (lllilliiiill << IlIiiliIIlii))) - (((((IIliIililllI << IlIiiliIIlii) - lllilliiiill) << iIIllliliiIl) - lllilliiiill) << (((((lllilliiiill << iIIllliliiIl) + lllilliiiill)) << IillillIlIll))) + (((((lllilliiiill << IillillIlIll) - lllilliiiill) << IillillIlIll) - iiIlIiIiIiIi) << ((((lllilliiiill << IillillIlIll) + lllilliiiill) << iIIllliliiIl) - lllilliiiill)) - (((lllilliiiill << iiIlIiIiIiIi) - iIIllliliiIl) << ((lllilliiiill << iiIlIiIiIiIi) - iIIllliliiIl)) + (((((iIIllliliiIl << IlIiiliIIlii) - lllilliiiill) << IlIiiliIIlii) - lllilliiiill) << ((((lllilliiiill << IillillIlIll) - lllilliiiill) << iIIllliliiIl) - iIIllliliiIl)) + (((iiIlIiIiIiIi << IillillIlIll) - lllilliiiill) << ((((iIIllliliiIl << IlIiiliIIlii) + lllilliiiill) << iIIllliliiIl) + iIIllliliiIl)) + (((((iIIllliliiIl << IlIiiliIIlii) + lllilliiiill) << iIIllliliiIl) - iIIllliliiIl) << ((iIIllliliiIl << IIliIililllI))) + (((lllilliiiill << lllilIlIiiIi) + lllilliiiill) << ((((iIIllliliiIl << IlIiiliIIlii) - lllilliiiill) << iIIllliliiIl) - lllilliiiill)) - ((((((lllilliiiill << iIIllliliiIl) + lllilliiiill)) << IillillIlIll) - iIIllliliiIl) << ((IIliIililllI << IillillIlIll) - iIIllliliiIl)) + (((((lllilliiiill << iIIllliliiIl) + lllilliiiill))) << ((((lllilliiiill << IillillIlIll) + lllilliiiill) << IlIiiliIIlii) + lllilliiiill)) + (((iiIlIiIiIiIi << IillillIlIll) + iIIllliliiIl) << ((iiIlIiIiIiIi << iIIllliliiIl))) + (((((iIIllliliiIl << IlIiiliIIlii) + lllilliiiill) << iIIllliliiIl) + lllilliiiill) << ((iIIllliliiIl << IillillIlIll))) + (((lllilliiiill << lllilIlIiiIi) + lllilliiiill) << ((IIliIililllI << iIIllliliiIl) - lllilliiiill)) - (((iIIllliliiIl << iIIllliliiIl) + lllilliiiill) << ((lllilliiiill << IIliIililllI) - lllilliiiill)) - (((((iIIllliliiIl << IlIiiliIIlii) - lllilliiiill) << IlIiiliIIlii) + lllilliiiill) << ((iIIllliliiIl << iIIllliliiIl) - lllilliiiill)) - (((iIIllliliiIl << IillillIlIll) - lllilliiiill) << ((lllilliiiill << IillillIlIll) - lllilliiiill)) - ((((iIIllliliiIl << IlIiiliIIlii) - lllilliiiill)) << ((IIliIililllI << lllilliiiill))) + (((lllilliiiill << IillillIlIll) + lllilliiiill) << lllilliiiill)        )    )(        *(lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl: lllilliiiill(lllilliiiill, IlIiiliIIlii, iIIllliliiIl))(            (lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl:                [IlIiiliIIlii(iIIllliliiIl[(lambda: lllilliiiill).__code__.co_nlocals])] +                lllilliiiill(lllilliiiill, IlIiiliIIlii, iIIllliliiIl[(lambda iiiiiiililli: iiiiiiililli).__code__.co_nlocals:]) if iIIllliliiIl else []            ),            lambda lllilliiiill: lllilliiiill.__code__.co_argcount,            (                lambda lllilliiiill: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll, IIliIililllI: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll, IIliIililllI, lllilIlIiiIi: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll, IIliIililllI, lllilIlIiiIi, iiIlIiIiIiIi: lllilliiiill,                lambda lllilliiiill, IlIiiliIIlii, iIIllliliiIl, IillillIlIll, IIliIililllI, lllilIlIiiIi, iiIlIiIiIiIi, IIlIlilIIIII: lllilliiiill            )        )    ).decode("utf-8")[1:-1])
    print(iIIiIliiillI(lambda IiiiiIlIIiiI: IiiiiIlIIiiI**(3 * (420 & 1984) + -2 * (420 & ~1984) + -1 * (~420 & 1984) + 3 * (420 ^ 1984) + -2 * (420 | 1984) + -1 * ~(420 ^ 1984) + 1 * ~1984 + -2 * -1), (-1 * (420 & 1984) + -1 * (420 & ~1984) + -1 * 420 + 1 * (~420 & 1984) + -1 * (420 ^ 1984) + -5 * ~(420 | 1984) + 2 * ~(420 ^ 1984) + 3 * ~1984 + -3 * -1)))
    iillIIillIll = iIllIIIiIliI((-1 * (1337 & ~31415) + 3 * 1337 + 1 * (~1337 & 31415) + -1 * (1337 ^ 31415) + 4 * ~(1337 | 31415) + -3 * ~(1337 ^ 31415) + -1 * ~31415 + -2 * -1))
    print(iillIIillIll.subclass)
    print(ilIiIlllIIli((1 * (747 & ~999) + -2 * 747 + -2 * (~747 & 999) + 2 * (747 | 999) + 1 * ~(747 | 999) + -1 * ~999 + -10 * -1)))
```