# 11 lines of no-package Python code that solve Matt Parker's Wordle challenge in 8 seconds - and 18 lines with numpy that do so in 4

(c) by Thomas Reichert

## License

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 3 of the License, or
(at your option) any later version.
This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.
You should have received a copy of the GNU General Public License
along with this program; if not, see http://www.gnu.org/licenses/.

## The challenge and my code

The goal of this jupyter notebook is to solve [Matt Parker's](https://standupmaths.com) Wordle challenge (see [his youtube video](https://www.youtube.com/watch?v=c33AZBnRHks&t=959s)) to find combinations of five english five letter words that contain 25 distinct letters **with my personal restriction that all has to happen in not significantly more than 10 lines of basic Python code using only one single thread and no additional packages that might speed up things**.

The motivation for this is that if one almost exclusively works with Python such as yours truly as a data scientist and certified actuary does, the fact that solving this problem is possible much faster by just implementing it in e.g. C++ doesn't really help a lot as one could not spontaneusly write code in it if one hasn't worked with it yet. Also, I want to showcase that - despite clearly not being the fastest language around - Python is a great choice to solve small problems like this one with *just a few humanly understandable lines of code* - given that one understands how list comprehensions work ;-) So Python it is with currently 12 lines that each fully fit into a Jupyter cell's default width and that in reality are 11 as tqdm is only imported and used to make the user feel better because 'something happens on the screen all the time'.

*If you run this notebook, please run the code itself first and patiently wait for about 11 seconds as only then the values in the next markdown section will appear properly*


```python
from tqdm import tqdm # only used to feel better because 'something happens on the screen all the time' ;-)
def cmb(x): # function that combines words recursively to tuples, triples, quadruples and quintuples
    return cmb([{'_'.join([kx, ky]): vx|vy for kx, vx in tqdm(x.pop(0).items()) for ky, vy in x[0].items()
                 if (vx&vy==0)}] + x[1:]) if (len(x) > 1) else x[0]
def val(f='words_alpha.txt', ana=True): #??ana = True drops anagrams which speeds up code by factor 2
    tmp = {w: sum(1<<(ord(c)-97) for c in w) for w in open(f).read().split('\n') if (len(w)==len(set(w))==5)}
    return {v: k for k, v in {v: k for k, v in tmp.items()}.items()} if ana else tmp
x = val(); valid = {str(n): {k: v for k, v in x.items() if (len(set(k) & set('aeiou'))<=n)} for n in range(4)}
valid.update({i: {k: v for k, v in valid['1'].items() if len(set(k) & set(j))>0}
              for i, j in {'a':'a','e':'e','i':'i','o':'o','u':'u','x':'eou','y':'eiu','z':'aeu'}.items()})
result = [k for d in [cmb([valid[v] for v in i]) for i in ['ueoia', '0xyz2', '00133']] for k in d.keys()]
open('result.csv', 'w').write('\n'.join(result:=set(','.join(sorted(k.split('_'))) for k in result)))
```

    100%|??????????????????????????????| 325/325 [00:00<00:00, 45415.59it/s]
    100%|??????????????????????????????| 26642/26642 [00:00<00:00, 84201.37it/s]
    100%|??????????????????????????????| 297146/297146 [00:03<00:00, 97994.44it/s]
    100%|??????????????????????????????| 110545/110545 [00:01<00:00, 73274.22it/s]
    100%|??????????????????????????????| 24/24 [00:00<00:00, 22620.97it/s]
    100%|??????????????????????????????| 2606/2606 [00:00<00:00, 33952.48it/s]
    100%|??????????????????????????????| 25392/25392 [00:00<00:00, 31793.70it/s]
    100%|??????????????????????????????| 10308/10308 [00:01<00:00, 7608.93it/s]
    100%|??????????????????????????????| 24/24 [00:00<00:00, 617566.23it/s]
    100%|??????????????????????????????| 10/10 [00:00<00:00, 16313.90it/s]
    100%|??????????????????????????????| 222/222 [00:00<00:00, 6486.31it/s]
    100%|??????????????????????????????| 4758/4758 [00:00<00:00, 6818.23it/s]

## What the code actually does
As the runtime of this algorithm for finding all possible combinations of five words with all distinct letters will be $O(n_1 * n_2 * n_3 * n_4 * n_5)$, we must keep our numbers of words $n_i$ for the first, second, etc. word as small as only possible. Hence the first thing we must do is to get rid of as many words as possible beforehand and divide our problem into as small as only possible subgroups.

For this, we drop all anagrams, meaning that out of e.g. *below, bowel and elbow* we only keep one as all three contain the same five letters. Then we can calculate an integer representation of each word, where in the binary number system a = 1, b = 10, c = 100, etc. Comparing the integer numbers resulting from these binary numbers is about 7 times faster than creating letter sets and comparing those, whereas dropping anagrams is about twice as fast as keeping them.

We can also make use of the fact that there are only 24 valid words such as crypt, glyph or nymph in the word list that contain none of the five vowels a, e, i, o, or u. This means that a combination of five words can only contain a word with two vowels if at least one of them contains none. As be seen easily (almost all of these either contain either an x or a y), the maximum number of words from that vowelless list that can be combined is two.

So we solve the problem in three parts, each of them starting from the smallest to the largest list of words to keep the number of comparisons on the way as low as possible:

* Five words where each contains none or one vowel. Here we can even split the word list by vowel to speed things up using 325 words that contain u, 370 words that contain e, 388 words that contain o, 411 words that contain i and 548 words that contain a.
* One out of 24 words that contains no vowel, three out of 2066 words that contain at most one and one out of 5348 words that contains at most two vowels. As proven in the appendix, one does not have to check the full one-vowel-list three times, it is enough to check the 1083 words that contain e, o or u first, then the 1106 words that contain e, i or u and then the 1243 words that contain a, e or u.
* Two out of 24 words that contain no vowel, one out of 2066 words that contains at most one and two out of 5960 words that contain at most three vowels.

and build everything together to find the 538 combinations without anagrams. While clearly being way off the results from the super fast precompiled languages, I was at least able to get a runtime of about 8 seconds on a M1 Macbook Pro and beat [Benjamin Paassen's graph theory approach](https://gitlab.com/bpaassen/five_clique), which was the first decently fast pure Python approach that I am aware of and served as my initial benchmark, by approximately a factor of more than 100 ;-) Meanwhile due to his comparison spreadsheet I had to figure out that there are already some Python solutions out there that are still faster than mine.

## Speeding up everything by allowing for vectorized numpy operations
Dropping the *no imported packages* constraint, I was able to speed up the code by roughly factor 2 using the well-known *numpy* package which allows for much faster vectorized operations instead of time costly for loops which are a known speed bottleneck in Python. This solution is conceptionally still the same algorithm, still single-threaded, below 20 lines of Python code and runs in 4 seconds. 

Will continue trying other optimization techniques - curious where the journey will take me ;-)


```python
from tqdm import tqdm; import numpy as np
def npcmb(x):  # function that combines words recursively to tuples, triples, quadruples and quintuples
    if len(x) > 1:
        kx, vx, ky, vy = list(x[0].keys()), list(x[0].values()), list(x[1].keys()), list(x[1].values())
        tmp = np.where(np.bitwise_and(np.array(vx).repeat(len(vy)).reshape(len(vx), len(vy)),
                                      np.array(vy).repeat(len(vx)).reshape(len(vy), len(vx)).transpose())==0)
        tmp = [{'_'.join([kx[tmp[0][i]], ky[tmp[1][i]]]): vx[tmp[0][i]] | vy[tmp[1][i]]
                       for i in range(len(tmp[0]))}] + x[2:]
        return npcmb(tmp)
    return x[0]
def val(f='words_alpha.txt', ana=True): #??ana = True drops anagrams which speeds up code by factor 2
    tmp = {w: sum(1<<(ord(c)-97) for c in w) for w in open(f).read().split('\n') if (len(w)==len(set(w))==5)}
    return {v: k for k, v in {v: k for k, v in tmp.items()}.items()} if ana else tmp
x = val(); valid = {str(n): {k: v for k, v in x.items() if (len(set(k) & set('aeiou'))<=n)} for n in range(4)}
valid.update({i: {k: v for k, v in valid['1'].items() if len(set(k) & set(j))>0}
              for i, j in {'a':'a','e':'e','i':'i','o':'o','u':'u','x':'eou','y':'eiu','z':'aeu'}.items()})
result = [k for d in [npcmb([valid[v] for v in i]) for i in ['ueoia', '0xyz2', '00133']] for k in d.keys()]
open('result.csv', 'w').write('\n'.join(result:=set(','.join(sorted(k.split('_'))) for k in result)))
```

## The result
Here are the 538 word combinations without anagrams:


```python
print(len(result))
print(sorted(result))
```

    538
    ['ampyx,bejig,fconv,hdqrs,klutz', 'ampyx,bewig,fconv,hdqrs,klutz', 'ampyx,bortz,chivw,fjeld,gunks', 'ampyx,crwth,fdubs,kling,vejoz', 'ampyx,flung,hdqrs,twick,vejoz', 'avick,benjy,fldxt,grosz,whump', 'backs,fldxt,grump,vejoz,whiny', 'backs,fldxt,humpy,vejoz,wring', 'backs,fldxt,murph,vejoz,wingy', 'backs,fldxt,ringy,vejoz,whump', 'backs,fldxt,rumpy,vejoz,whing', 'backy,fldxt,grump,vejoz,whins', 'backy,fldxt,murph,vejoz,wings', 'backy,fldxt,rings,vejoz,whump', 'backy,fldxt,rumps,vejoz,whing', 'backy,fldxt,sumph,vejoz,wring', 'baken,chivw,fldxt,grosz,jumpy', 'bangs,fldxt,humpy,vejoz,wrick', 'bangs,fldxt,murph,vejoz,wicky', 'bangs,fldxt,ricky,vejoz,whump', 'bangs,fldxt,rumpy,vejoz,whick', 'bangy,crump,fldxt,vejoz,whisk', 'bangy,fldxt,murph,vejoz,wicks', 'bangy,fldxt,ricks,vejoz,whump', 'bangy,fldxt,rumps,vejoz,whick', 'bangy,fldxt,sumph,vejoz,wrick', 'bangy,flump,hdqrs,twick,vejoz', 'banks,fldxt,gyric,vejoz,whump', 'banky,crump,fldxt,vejoz,whigs', 'bargh,fldxt,numps,vejoz,wicky', 'barks,chump,fldxt,vejoz,wingy', 'barks,fldxt,gynic,vejoz,whump', 'barms,fldxt,pungy,vejoz,whick', 'barmy,fldxt,pucks,vejoz,whing', 'barmy,fldxt,spung,vejoz,whick', 'bawke,fldxt,gconv,jimpy,qursh', 'bawke,fultz,gconv,hdqrs,jimpy', 'becks,fldxt,ginzo,jarvy,whump', 'becks,fldxt,jumpy,vizor,whang', 'becks,fldxt,jumpy,voraz,whing', 'becks,fldxt,rajiv,whump,zygon', 'becks,frowl,japyx,vingt,zhmud', 'becks,fultz,japyx,mordv,whing', 'becky,fldxt,jumps,vizor,whang', 'becky,fldxt,jumps,voraz,whing', 'behav,fldxt,jumps,wrick,zygon', 'bejan,fldxt,grosz,vicky,whump', 'bejig,chump,fldxt,knyaz,vrows', 'bejig,fldxt,knyaz,mowch,supvr', 'bejig,fldxt,nymph,quack,vrows', 'bemix,fultz,gconv,hdqrs,pawky', 'bench,fldxt,gawks,jumpy,vizor', 'bench,fldxt,gawky,jumps,vizor', 'bench,fldxt,grosz,jimpy,quawk', 'benjy,chimp,fldxt,grosz,quawk', 'benjy,chowk,fldxt,gramp,squiz', 'benjy,chump,fldxt,gawks,vizor', 'benjy,fldxt,gizmo,supvr,whack', 'benjy,fldxt,grosz,quick,whamp', 'benjy,fldxt,gucks,vizor,whamp', 'benjy,fldxt,oghuz,vamps,wrick', 'bevor,fldxt,jacks,whump,zingy', 'bevor,fldxt,jacky,whump,zings', 'bevor,fldxt,jumps,whack,zingy', 'bevor,fldxt,jumpy,whack,zings', 'bewig,fconv,hdqrs,japyx,klutz', 'bewig,flock,hdqrs,japyx,muntz', 'bezan,fldxt,grovy,jumps,whick', 'bhang,crump,fldxt,skiwy,vejoz', 'bhang,fldxt,rumps,vejoz,wicky', 'bhang,fldxt,rumpy,vejoz,wicks', 'bhang,fldxt,spumy,vejoz,wrick', 'bichy,fldxt,grump,swank,vejoz', 'bichy,fldxt,gunks,vejoz,wramp', 'bigam,expwy,fconv,hdqrs,klutz', 'bilks,fconv,grewt,japyx,zhmud', 'bingy,chump,fldxt,vejoz,warks', 'bingy,crump,fldxt,hawks,vejoz', 'bingy,fldxt,mucks,vejoz,wharp', 'bingy,fldxt,murph,vejoz,wacks', 'bingy,fldxt,racks,vejoz,whump', 'bingy,fldxt,rucks,vejoz,whamp', 'bingy,fldxt,rumps,vejoz,whack', 'bingy,fldxt,shuck,vejoz,wramp', 'bingy,fldxt,sumph,vejoz,wrack', 'birch,fldxt,gawky,numps,vejoz', 'birch,fldxt,kyung,swamp,vejoz', 'birch,fldxt,mawks,pungy,vejoz', 'birch,fldxt,mawky,spung,vejoz', 'bleck,frows,japyx,vingt,zhmud', 'blitz,fconv,gawky,hdqrs,pumex', 'blitz,fconv,gryph,judex,mawks', 'block,fremt,japyx,vughs,windz', 'bocks,fldxt,javer,whump,zingy', 'bocks,flegm,japyx,thruv,windz', 'bongs,chivw,fldxt,jumpy,karez', 'bongs,chivw,fremd,japyx,klutz', 'bonze,fldxt,gravy,jumps,whick', 'bonze,fldxt,jimpy,vughs,wrack', 'bortz,chivw,dunks,flegm,japyx', 'bovld,freck,japyx,muntz,whigs', 'bowge,flick,hdqrs,japyx,muntz', 'bowls,freck,japyx,vingt,zhmud', 'brack,fldxt,humpy,vejoz,wings', 'brack,fldxt,jives,whump,zygon', 'brack,fldxt,jowpy,vughs,zemni', 'brack,fldxt,mungy,vejoz,whisp', 'brack,fldxt,pungy,vejoz,whims', 'brack,fldxt,spumy,vejoz,whing', 'brack,fldxt,sumph,vejoz,wingy', 'brahm,fldxt,picky,swung,vejoz', 'brahm,fldxt,pucks,vejoz,wingy', 'brahm,fldxt,pungy,vejoz,wicks', 'brahm,fldxt,spung,vejoz,wicky', 'braky,chimp,fldxt,swung,vejoz', 'braky,chump,fldxt,vejoz,wings', 'brawn,fldxt,kopje,vughs,zymic', 'brawn,fldxt,pigmy,shuck,vejoz', 'braws,chimp,fldxt,kyung,vejoz', 'braws,chunk,fldxt,pigmy,vejoz', 'braws,fldxt,gucki,nymph,vejoz', 'braws,fldxt,nymph,quick,vejoz', 'breck,fldxt,jowpy,nizam,vughs', 'breck,japyx,vingt,wolfs,zhmud', 'breva,chowk,fldxt,jumps,zingy', 'breva,chowk,fldxt,jumpy,zings', 'breva,fldxt,jocks,whump,zingy', 'breva,fldxt,jumps,whick,zygon', 'brevi,fldxt,jacks,whump,zygon', 'brevi,fldxt,jumps,whack,zygon', 'brews,flock,japyx,vingt,zhmud', 'brick,fldxt,humpy,swang,vejoz', 'brick,fldxt,mungy,vejoz,whaps', 'brick,fldxt,nymph,quags,vejoz', 'brick,fldxt,nymph,squaw,vejoz', 'brick,fldxt,phyma,swung,vejoz', 'brick,fldxt,pungy,vejoz,whams', 'brick,fldxt,spumy,vejoz,whang', 'brick,fldxt,vejoz,whump,yangs', 'brigs,chump,fldxt,vejoz,wanky', 'brigs,fldxt,mawky,punch,vejoz', 'brigs,fldxt,munch,pawky,vejoz', 'brigs,fldxt,nymph,quack,vejoz', 'brigs,fldxt,nymph,quawk,vejoz', 'brims,chung,fldxt,pawky,vejoz', 'brims,fldxt,gawky,punch,vejoz', 'brims,fldxt,pungy,vejoz,whack', 'bring,chums,fldxt,pawky,vejoz', 'bring,fldxt,humpy,vejoz,wacks', 'bring,fldxt,mucky,vejoz,whaps', 'bring,fldxt,psych,quawk,vejoz', 'bring,fldxt,spumy,vejoz,whack', 'bring,fldxt,sumph,vejoz,wacky', 'bring,fldxt,vejoz,whamp,yucks', 'bring,fldxt,vejoz,whump,yacks', 'brink,chump,fldxt,gawsy,vejoz', 'brins,chump,fldxt,gawky,vejoz', 'briny,chump,fldxt,gawks,vejoz', 'briny,fldxt,gucks,vejoz,whamp', 'brisk,cangy,fldxt,vejoz,whump', 'brisk,fldxt,gconv,jazey,whump', 'brock,fldxt,jimpy,vughs,wanze', 'brock,flews,japyx,vingt,zhmud', 'brock,japyx,seqwl,vingt,zhmud', 'bronk,chivw,fldxt,gazes,jumpy', 'brows,fleck,japyx,vingt,zhmud', 'bruja,chivw,fldxt,skemp,zygon', 'brusk,campy,fldxt,vejoz,whing', 'brusk,champ,fldxt,vejoz,wingy', 'brusk,chawn,fldxt,pigmy,vejoz', 'brusk,fldxt,gynic,vejoz,whamp', 'bryum,fldxt,gawks,pinch,vejoz', 'bryum,fldxt,pings,vejoz,whack', 'bryum,fldxt,spack,vejoz,whing', 'bryum,fldxt,spang,vejoz,whick', 'bryum,fldxt,spick,vejoz,whang', 'bucks,fldxt,gramp,vejoz,whiny', 'bucks,fldxt,hying,vejoz,wramp', 'bucks,fldxt,javer,whomp,zingy', 'bucks,fldxt,mingy,vejoz,wharp', 'bucks,fldxt,phyma,vejoz,wring', 'bucks,fldxt,prahm,vejoz,wingy', 'bucks,fldxt,primy,vejoz,whang', 'bucks,fldxt,ringy,vejoz,whamp', 'bucky,fldxt,gramp,vejoz,whins', 'bucky,fldxt,grimp,shawn,vejoz', 'bucky,fldxt,javer,whomp,zings', 'bucky,fldxt,phasm,vejoz,wring', 'bucky,fldxt,prahm,vejoz,wings', 'bucky,fldxt,prang,vejoz,whims', 'bucky,fldxt,prism,vejoz,whang', 'bucky,fldxt,rings,vejoz,whamp', 'bucky,fldxt,singh,vejoz,wramp', 'bucky,fldxt,sparm,vejoz,whing', 'bucky,flimp,hdqrs,twang,vejoz', 'bumph,crang,fldxt,skiwy,vejoz', 'bumph,crink,fldxt,gawsy,vejoz', 'bumph,fldxt,gawky,scrin,vejoz', 'bumph,fldxt,gconv,jerky,swazi', 'bumph,fldxt,ginzo,jacks,wyver', 'bumph,fldxt,gnars,vejoz,wicky', 'bumph,fldxt,gowks,javer,zincy', 'bumph,fldxt,gracy,vejoz,winks', 'bumph,fldxt,gravy,jocks,wizen', 'bumph,fldxt,grosz,jacky,vinew', 'bumph,fldxt,grosz,njave,wicky', 'bumph,fldxt,grovy,jacks,wizen', 'bumph,fldxt,gynic,vejoz,warks', 'bumph,fldxt,gyric,swank,vejoz', 'bumph,fldxt,jacko,wyver,zings', 'bumph,fldxt,jacks,vower,zingy', 'bumph,fldxt,jacks,wrive,zygon', 'bumph,fldxt,jacky,vower,zings', 'bumph,fldxt,javer,wicks,zygon', 'bumph,fldxt,jives,wrack,zygon', 'bumph,fldxt,jocks,waver,zingy', 'bumph,fldxt,jocks,wyver,zigan', 'bumph,fldxt,racks,vejoz,wingy', 'bumph,fldxt,rangy,vejoz,wicks', 'bumph,fldxt,ricky,swang,vejoz', 'bumph,fldxt,rings,vejoz,wacky', 'bumph,fldxt,ringy,vejoz,wacks', 'bumph,fldxt,vejoz,wrick,yangs', 'bumph,fldxt,vejoz,wring,yacks', 'bumps,ching,fldxt,rawky,vejoz', 'bumps,chivw,fldxt,jerky,zogan', 'bumps,chivw,fldxt,jorge,knyaz', 'bumps,chowk,fldxt,javer,zingy', 'bumps,fldxt,gawky,rinch,vejoz', 'bumps,fldxt,gyric,vejoz,whank', 'bumps,fldxt,hacky,vejoz,wring', 'bumps,fldxt,hicky,vejoz,wrang', 'bumps,fldxt,hying,vejoz,wrack', 'bumps,fldxt,jahve,wrick,zygon', 'bumps,fldxt,javer,whick,zygon', 'bumps,fldxt,karch,vejoz,wingy', 'bumps,fldxt,rangy,vejoz,whick', 'bumps,fldxt,ricky,vejoz,whang', 'bumps,fldxt,ringy,vejoz,whack', 'bumpy,ching,fldxt,vejoz,warks', 'bumpy,chirk,fldxt,swang,vejoz', 'bumpy,chivw,fldxt,jerks,zogan', 'bumpy,chowk,fldxt,javer,zings', 'bumpy,crang,fldxt,vejoz,whisk', 'bumpy,crank,fldxt,vejoz,whigs', 'bumpy,flang,hdqrs,twick,vejoz', 'bumpy,fldxt,gawks,rinch,vejoz', 'bumpy,fldxt,gnars,vejoz,whick', 'bumpy,fldxt,grosz,njave,whick', 'bumpy,fldxt,karch,vejoz,wings', 'bumpy,fldxt,kings,vejoz,warch', 'bumpy,fldxt,racks,vejoz,whing', 'bumpy,fldxt,ricks,vejoz,whang', 'bumpy,fldxt,rings,vejoz,whack', 'bumpy,fldxt,shack,vejoz,wring', 'bumpy,fldxt,shang,vejoz,wrick', 'bumpy,fldxt,shick,vejoz,wrang', 'bumpy,fldxt,singh,vejoz,wrack', 'bumpy,flick,hdqrs,twang,vejoz', 'bunch,fldxt,gawks,primy,vejoz', 'bunch,fldxt,gawky,prism,vejoz', 'bunch,fldxt,gimps,rawky,vejoz', 'bunch,fldxt,gramp,skiwy,vejoz', 'bunch,fldxt,gripy,mawks,vejoz', 'bunch,fldxt,mawky,sprig,vejoz', 'bunch,fldxt,pigmy,vejoz,warks', 'bungs,chimp,fldxt,rawky,vejoz', 'bungs,chirm,fldxt,pawky,vejoz', 'bungs,crimp,fldxt,hawky,vejoz', 'bungs,fldxt,hicky,vejoz,wramp', 'bungs,fldxt,mawky,prich,vejoz', 'bungs,fldxt,micky,vejoz,wharp', 'bungs,fldxt,phyma,vejoz,wrick', 'bungs,fldxt,prahm,vejoz,wicky', 'bungs,fldxt,primy,vejoz,whack', 'bungs,fldxt,ricky,vejoz,whamp', 'bungy,chimp,fldxt,vejoz,warks', 'bungy,chirk,fldxt,swamp,vejoz', 'bungy,cramp,fldxt,vejoz,whisk', 'bungy,crimp,fldxt,hawks,vejoz', 'bungy,fldxt,mawks,prich,vejoz', 'bungy,fldxt,micks,vejoz,wharp', 'bungy,fldxt,phasm,vejoz,wrick', 'bungy,fldxt,prahm,vejoz,wicks', 'bungy,fldxt,prick,vejoz,whams', 'bungy,fldxt,prism,vejoz,whack', 'bungy,fldxt,ricks,vejoz,whamp', 'bungy,fldxt,shick,vejoz,wramp', 'bungy,fldxt,skimp,vejoz,warch', 'bungy,fldxt,sparm,vejoz,whick', 'bungy,hdqrs,lampf,twick,vejoz', 'bunks,fldxt,gyric,vejoz,whamp', 'bunks,fldxt,pigmy,vejoz,warch', 'burez,chowk,fldxt,jimpy,vangs', 'burez,fldxt,gconv,hawks,jimpy', 'burez,fldxt,jacks,vying,whomp', 'burez,fldxt,jocks,vying,whamp', 'burga,fldxt,nymph,vejoz,wicks', 'burgh,campy,fldxt,vejoz,winks', 'burgh,cawny,fldxt,skimp,vejoz', 'burgh,fldxt,micky,spawn,vejoz', 'burgh,fldxt,nicky,swamp,vejoz', 'burns,chimp,fldxt,gawky,vejoz', 'burns,fldxt,pigmy,vejoz,whack', 'burps,ching,fldxt,mawky,vejoz', 'burps,fldxt,mangy,vejoz,whick', 'burps,fldxt,mckay,vejoz,whing', 'burps,fldxt,micky,vejoz,whang', 'burps,fldxt,mingy,vejoz,whack', 'busky,champ,fldxt,vejoz,wring', 'busky,chawn,fldxt,grimp,vejoz', 'busky,chimp,fldxt,vejoz,wrang', 'busky,ching,fldxt,vejoz,wramp', 'busky,cramp,fldxt,vejoz,whing', 'busky,crimp,fldxt,vejoz,whang', 'busky,fldxt,gramp,vejoz,winch', 'carby,fldxt,kings,vejoz,whump', 'carvy,fldxt,jinks,uzbeg,whomp', 'champ,fldxt,kirby,swung,vejoz', 'champ,fldxt,rugby,vejoz,winks', 'chank,fldxt,jimpy,uzbeg,vrows', 'chank,fldxt,jowpy,mirvs,uzbeg', 'chawn,fldxt,gumby,skirp,vejoz', 'chawn,fldxt,rugby,skimp,vejoz', 'cheng,fldxt,jimpy,uzbak,vrows', 'cheng,fldxt,jowpy,mirvs,uzbak', 'chimb,expwy,fjord,klutz,vangs', 'chimb,fldxt,gawky,spurn,vejoz', 'chimb,fldxt,kyung,vejoz,wraps', 'chimb,fldxt,parky,swung,vejoz', 'chimb,fldxt,pawky,rungs,vejoz', 'chimb,fldxt,pungy,vejoz,warks', 'chimb,fldxt,rawky,spung,vejoz', 'chimb,fldxt,sprug,vejoz,wanky', 'chimp,fldxt,gawby,knurs,vejoz', 'chimp,fldxt,gawks,runby,vejoz', 'chimp,fldxt,gowns,jarvy,uzbek', 'chimp,fldxt,grubs,vejoz,wanky', 'chimp,fldxt,jarvy,swonk,uzbeg', 'chimp,fldxt,quawk,verbs,zygon', 'chimp,fldxt,rugby,swank,vejoz', 'chirk,fldxt,gawby,numps,vejoz', 'chirk,fldxt,gumby,spawn,vejoz', 'chirm,fldxt,gawby,spunk,vejoz', 'chirm,fldxt,jowpy,uzbek,vangs', 'chivw,enzym,fldxt,jakob,sprug', 'chivw,expdt,flamb,grosz,junky', 'chivw,expdt,flank,grosz,jumby', 'chivw,expdt,furzy,jambs,klong', 'chivw,expdt,jumby,klong,zarfs', 'chivw,fjord,glaky,muntz,pbxes', 'chivw,fjord,klutz,mangy,pbxes', 'chivw,fldxt,graze,jumby,knosp', 'chivw,fldxt,graze,jumpy,knobs', 'chivw,fldxt,grebo,jumps,knyaz', 'chivw,fldxt,grosz,jambe,punky', 'chivw,fldxt,grosz,jumby,pekan', 'chivw,fldxt,grump,jazey,knobs', 'chivw,fldxt,grype,junks,zambo', 'chivw,fldxt,jambs,puker,zygon', 'chivw,fldxt,jerks,pungy,zambo', 'chivw,fldxt,jerky,spung,zambo', 'chivw,fldxt,jumba,perks,zygon', 'chivw,fldxt,jumby,karez,spong', 'chivw,fldxt,jumby,perks,zogan', 'chivw,fldxt,jumps,kebar,zygon', 'chivw,fldxt,jumpy,krebs,zogan', 'chomp,fldxt,jarvy,uzbeg,winks', 'chomp,fldxt,jarvy,uzbek,wings', 'chomp,fldxt,quawk,verbs,zingy', 'chomp,fldxt,quawk,verby,zings', 'chowk,fldxt,gravy,jumps,zineb', 'chowk,fldxt,jambe,supvr,zingy', 'chowk,fldxt,jumby,prize,vangs', 'chowk,fldxt,jumby,verpa,zings', 'chowk,fldxt,jumps,verby,zigan', 'chowk,fldxt,jumps,vying,zebra', 'chowk,fldxt,jumpy,verbs,zigan', 'chump,fldxt,gawby,rinks,vejoz', 'chump,fldxt,gowks,jarvy,zineb', 'chump,fldxt,jakob,wyver,zings', 'chump,fldxt,kirby,swang,vejoz', 'chums,fldxt,gawby,prink,vejoz', 'chung,fldxt,kirby,swamp,vejoz', 'chunk,fldxt,gawby,prism,vejoz', 'churm,fldxt,gawby,spink,vejoz', 'comps,fldxt,jarvy,uzbek,whing', 'crank,fldxt,gumby,vejoz,whisp', 'crawm,fldxt,qophs,uzbek,vying', 'cribs,fldxt,kyang,vejoz,whump', 'cribs,fldxt,kyung,vejoz,whamp', 'cribs,fldxt,nymph,quawk,vejoz', 'crimp,fldxt,gawby,hunks,vejoz', 'crink,fldxt,gawby,sumph,vejoz', 'crink,fldxt,gumby,vejoz,whaps', 'crumb,fldxt,gipsy,vejoz,whank', 'crumb,fldxt,hawky,pings,vejoz', 'crumb,fldxt,kaphs,vejoz,wingy', 'crumb,fldxt,kyang,vejoz,whisp', 'crumb,fldxt,pawky,singh,vejoz', 'crumb,fldxt,spiky,vejoz,whang', 'crump,fldxt,gawby,knish,vejoz', 'crwth,gambs,kylix,pfund,vejoz', 'curby,fldxt,gimps,vejoz,whank', 'curby,fldxt,kings,vejoz,whamp', 'curby,fldxt,skimp,vejoz,whang', 'dhikr,expwy,fultz,gconv,jambs', 'dumbs,fritz,gconv,japyx,whelk', 'dwarf,glyph,jocks,muntz,vibex', 'expdt,furzy,gconv,jambs,whilk', 'expdt,gconv,jumby,whilk,zarfs', 'exptl,fconv,gawby,hdqrs,mujik', 'exptl,gconv,hdqrs,jumby,kafiz', 'exptl,gconv,hdqrs,jumby,wakif', 'expwy,flack,hdqrs,jumbo,vingt', 'expwy,flock,hdqrs,jumba,vingt', 'fangy,hdqrs,plumb,twick,vejoz', 'fcomp,gawby,hdqrs,klutz,vixen', 'fcomp,hdqrs,junky,vibex,waltz', 'fcomp,hdqrs,kyung,vibex,waltz', 'fjord,glyph,muntz,vibex,wacks', 'fjord,gucks,nymph,vibex,waltz', 'flack,hdqrs,jowpy,muntz,vibex', 'flamb,hdqrs,pungy,twick,vejoz', 'fldxt,gamps,runby,vejoz,whick', 'fldxt,ganch,jimpy,uzbek,vrows', 'fldxt,ganch,jowpy,mirvs,uzbek', 'fldxt,gawby,jumps,kench,vizor', 'fldxt,gawby,kinch,rumps,vejoz', 'fldxt,gawby,munch,skirp,vejoz', 'fldxt,gawby,murks,pinch,vejoz', 'fldxt,gawby,murph,snick,vejoz', 'fldxt,gawby,punch,smirk,vejoz', 'fldxt,gawby,runch,skimp,vejoz', 'fldxt,gawks,nymph,urbic,vejoz', 'fldxt,gawky,numbs,prich,vejoz', 'fldxt,gconv,hawks,jumby,prize', 'fldxt,gconv,herbs,jimpy,quawk', 'fldxt,gconv,jerky,sabzi,whump', 'fldxt,gconv,jerky,saqib,whump', 'fldxt,gconv,jerky,squib,whamp', 'fldxt,gconv,jerky,squiz,whamp', 'fldxt,gconv,jimpy,uzbak,wersh', 'fldxt,gconv,jizya,krebs,whump', 'fldxt,gconv,jumby,karez,whisp', 'fldxt,gconv,jumpy,whisk,zebra', 'fldxt,ghbor,jumps,vicky,wanze', 'fldxt,gibus,nymph,vejoz,wrack', 'fldxt,gimps,jarvy,nowch,uzbek', 'fldxt,gimps,runby,vejoz,whack', 'fldxt,ginzo,jacks,verby,whump', 'fldxt,ginzo,jacky,verbs,whump', 'fldxt,ginzo,jumby,pshav,wreck', 'fldxt,ginzo,jumps,verby,whack', 'fldxt,ginzo,jumpy,verbs,whack', 'fldxt,given,jumby,qophs,wrack', 'fldxt,grabs,nicky,vejoz,whump', 'fldxt,grabs,nymph,quick,vejoz', 'fldxt,graph,numbs,vejoz,wicky', 'fldxt,grapy,numbs,vejoz,whick', 'fldxt,gravy,jocks,whump,zineb', 'fldxt,grimp,quawk,synch,vejoz', 'fldxt,grimp,subch,vejoz,wanky', 'fldxt,griph,numbs,vejoz,wacky', 'fldxt,gripy,numbs,vejoz,whack', 'fldxt,grosz,jumby,paven,whick', 'fldxt,grovy,jacks,whump,zineb', 'fldxt,grovy,jumps,whack,zineb', 'fldxt,grubs,mawky,pinch,vejoz', 'fldxt,grubs,nicky,vejoz,whamp', 'fldxt,gryph,jambs,quick,woven', 'fldxt,gryph,manqu,vejoz,wicks', 'fldxt,gryph,minbu,vejoz,wacks', 'fldxt,gryph,njave,quick,wombs', 'fldxt,gucks,jarvy,whomp,zineb', 'fldxt,gumby,kinch,vejoz,wraps', 'fldxt,gumby,njave,qophs,wrick', 'fldxt,gumby,parch,vejoz,winks', 'fldxt,gumby,pinch,vejoz,warks', 'fldxt,gumby,pirns,vejoz,whack', 'fldxt,gumby,prawn,shick,vejoz', 'fldxt,gumby,prich,swank,vejoz', 'fldxt,gumby,prick,shawn,vejoz', 'fldxt,gumby,prink,schwa,vejoz', 'fldxt,gumby,scrip,vejoz,whank', 'fldxt,gumby,snick,vejoz,wharp', 'fldxt,gumby,spark,vejoz,winch', 'fldxt,gumby,spink,vejoz,warch', 'fldxt,jacko,verbs,whump,zingy', 'fldxt,jacko,verby,whump,zings', 'fldxt,jambe,qophs,vicky,wrung', 'fldxt,jambe,supvr,whick,zygon', 'fldxt,jarvy,mowch,pings,uzbek', 'fldxt,jarvy,mowch,spink,uzbeg', 'fldxt,jarvy,nowch,skimp,uzbeg', 'fldxt,jarvy,snick,uzbeg,whomp', 'fldxt,jerks,vocab,whump,zingy', 'fldxt,jerky,vocab,whump,zings', 'fldxt,jimpy,schav,uzbek,wrong', 'fldxt,jocks,verby,whump,zigan', 'fldxt,jocks,vying,whump,zebra', 'fldxt,johns,uzbeg,vicky,wramp', 'fldxt,jumba,qophs,vying,wreck', 'fldxt,jumba,qophs,wreck,zingy', 'fldxt,jumbo,pshav,wreck,zingy', 'fldxt,jumby,navig,qophs,wreck', 'fldxt,jumby,prove,whack,zings', 'fldxt,jumby,qophs,vegan,wrick', 'fldxt,jumby,qophs,wreck,zigan', 'fldxt,jumby,speck,vizor,whang', 'fldxt,jumby,speck,voraz,whing', 'fldxt,jumps,verby,whick,zogan', 'fldxt,jumpy,verbs,whick,zogan', 'fldxt,kinch,rugby,swamp,vejoz', 'fldxt,kumbi,psych,vejoz,wrang', 'fldxt,mawks,pinch,rugby,vejoz', 'fldxt,namby,sprug,vejoz,whick', 'fldxt,nymph,quags,vejoz,wrick', 'fldxt,nymph,squab,vejoz,wrick', 'fldxt,nymph,squib,vejoz,wrack', 'fldxt,picky,rhumb,swang,vejoz', 'fldxt,pigmy,scrub,vejoz,whank', 'fldxt,pings,rhumb,vejoz,wacky', 'fldxt,quack,verbs,whomp,zingy', 'fldxt,quack,verby,whomp,zings', 'fldxt,quick,verbs,whamp,zygon', 'fldxt,rhumb,spack,vejoz,wingy', 'fldxt,rhumb,spang,vejoz,wicky', 'fldxt,rugby,snick,vejoz,whamp', 'flock,hdqrs,jumpy,twang,vibex', 'flong,japyx,twick,verbs,zhmud', 'flong,jarvy,pbxes,twick,zhmud', 'flowk,hdqrs,imcnt,japyx,uzbeg', 'frack,jowly,pbxes,vingt,zhmud', 'frock,japyx,seqwl,vingt,zhmud', 'frowl,jacky,pbxes,vingt,zhmud', 'fultz,jacky,mordv,pbxes,whing', 'glack,hdqrs,jowpy,muntz,vibex']


## Appendix
### Numbers of words that contain only the allowed vowels and no others by vowel or vowel triple
These numbers are important because one should start with the vowel or vowel triple that has the lowest number of words to keep the number of n-word combinations as small as possible when progressing n from 1 to 5.

Overall there are 5 vowels (which we will use all) and 10 possible vowel triples (of wich we will use only 3).


```python
{i: len({k: v for k, v in valid['1'].items() if len(set(k) & set(i))>0})
 for i in ['a','e','i','o','u','aei','aeo','aeu','aio','aiu', 'aou','eio','eiu','eou','iou']}
```

    {'a': 548,
     'e': 370,
     'i': 411,
     'o': 388,
     'u': 325,
     'aei': 1329,
     'aeo': 1306,
     'aeu': 1243,
     'aio': 1347,
     'aiu': 1284,
     'aou': 1261,
     'eio': 1169,
     'eiu': 1106,
     'eou': 1083,
     'iou': 1124}



### Proof that the 3 selected vowel triples allow all 10 vowel combinations of 3 one-vowel words


```python
import itertools
len(sorted(list(set(''.join(sorted(i)) for i in list(
    itertools.product(*[list('eou'), list('eiu'), list('aeu')]))if len(set(i))==3))))
```

##  Final thoughts on my approach and my favourite solution by somebody else
### My own approach
Even though this approach clearly is no speed champ, my hope is that this *vowel based reduction of possible word combinations that need to be checked* can maybe help somebody else's super fast algorithm to even save a bit more time ;-)

What I found a surprise in the progress is that only a few out of these 538  combinations contain none of those 24 non-vowel words. Unfortunately finding exactly those few combinations is currently the speed bottleneck of my approach. Originally I thought that combinations with non-vowel words would be the exception rather than the rule, but I had to learn that using one of them opens the chance for another one to contain one word with two vowels, which are 2.6 times more frequent than words with only one vowel.

### Appreciation of other solutions
In the end, for me working as a data scientist coding is always a compromise between computational speed and the time and effort required for writing code or maintaining code written by somebody else. There are many situations where a blazing fast solution consisting of a lot of complex lines of code is optimal, but there alre also others where this code can be inferior to a handful lines of easily understandable and maintainable code despite the latter runs several times longer. That???s exactly what I love about Python ??? if speed at any cost is not your prime requirement you can be extremely code efficient. And if it is, it forces you to think about clever algorithm implementations instead of just relying on the intrinsic speed advantages of a programming language such as C++ ;-)

Thad said, I really like [Stefan Pochmann's](https://replit.com/@pochmann/5words538?v=1) solution for its combination of code brevity and runtime speed. Imho he solved this challenge way better than I did. Not only does his solution only take a third of my no-numpy solution's runtime, it is also the only other solution that I am aware of that not only came close to mine with respect to code brevity but even clearly beat me there, too.

Here's a slight adaption of Stefan's code, the only mentionable changes I made are that the solution now is actually returned as an object that could be processed further instead of just being printed out, and a switch for deciding whether one wants to drop anagrams beforehand or not. So all kudos for the algorithm itself should still go entirely to Stefan!


```python
def solve(words, chset=set('abcdefghijklmnopqrstuvwxyz'), used=[], result=[], drop_ana=True):
    if not chset: return result + [[*(u for u in used if len(u)==5)]]
    if drop_ana: words = [*{frozenset(w): w for w in words}.values()]
    for w in words+(len(chset)%5)*[c:=min(chset, key=''.join(words).count)]:
        if c in w: result = solve([*filter({*w}.isdisjoint, words)], chset-{*w}, used+[w], result, False)
    return result
solution = solve([w for w in open('words_alpha.txt').read().split() if 5==len(w)==len(set(w))])
```


```python
print(len(solution))
```

    538