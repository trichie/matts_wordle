# Matt Parker's Wordle challenge - the *pythonic vowel approach*

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

## The code
*If you run this notebook, please run the code itself first and patiently wait for a few minutes as only then the values in the next markdown section will appear properly ;-)*


```python
from tqdm import tqdm # only used to feel better because 'something happens on the screen all the time' ;-)
def cmb(x, l=10): # function that combines words recursively to tuples, triples, quadruples and quintuples
    return cmb([{'_'.join([kx, ky]): vx|vy for kx, vx in tqdm(x.pop(0).items()) for ky, vy in x[0].items()
                 if (len(vx|vy)==l)}] + x[1:], l+5) if (len(x) > 1) else x[0]
valid = {w: s for w in open('words_alpha.txt').read().split('\n') if (len(w)==5) and (len(s:=set(w))==5)} 
valid = {n: {k: v for k,v in valid.items() if (len(v & set(vwl:='aeiou'))<=n)} for n in [0, 1, 2, 3]}
vowels = {vw: {k: v for k,v in valid[1].items() if len(set(i for i in vwl if i!=vw)&v)<=0} for vw in vwl} 
result = {**(zero:=cmb([vowels[v] for v in 'uieoa'])), **(two:=cmb([valid[n] for n in [0, 0, 1, 3, 3]])),
          **(one:=cmb([valid[n] for n in [0, 1, 1, 1, 2]]))} # yes the order in each cmb matters speedwise
open('result.csv', 'w').write('\n'.join(result:=set(','.join(sorted(k.split('_'))) for k in result.keys())))
```

    100%|██████████| 431/431 [00:00<00:00, 3824.62it/s]
    100%|██████████| 47924/47924 [00:05<00:00, 8631.54it/s] 
    100%|██████████| 497833/497833 [01:09<00:00, 7130.84it/s]
    100%|██████████| 167055/167055 [00:33<00:00, 4932.52it/s]
    100%|██████████| 26/26 [00:00<00:00, 132505.35it/s]
    100%|██████████| 10/10 [00:00<00:00, 1592.07it/s]
    100%|██████████| 282/282 [00:00<00:00, 351.69it/s]
    100%|██████████| 7324/7324 [00:19<00:00, 384.29it/s]
    100%|██████████| 26/26 [00:00<00:00, 1314.54it/s]
    100%|██████████| 6546/6546 [00:03<00:00, 1755.71it/s]
    100%|██████████| 128290/128290 [01:37<00:00, 1314.76it/s]
    100%|██████████| 73434/73434 [02:50<00:00, 430.36it/s]


## What the code actually does

The goal of this jupyter notebook is to solve Matt Parker's Wordle challenge **with my personal restriction that all has to happen in 10 lines of basic Python code using only one single thread and no additional packages that might speed up things**. The motivation for this is that if one almost exclusively works with Python such as yours truly as a data scientist and certified actuary does, the fact that solving this problem is possible more than 1000 times faster by just implementing it in e.g. C++ doesn't really help a lot as one could not spontaneusly write code in it. So Python it is ;-)

The only package being imported is tqdm which doesn't help the algorithm at all but just shows progress to make us humans feel better because 'something happens on the screen all the time' ;-). While clearly being way off the results from the super fast precompiled languages, I was at least able to get a runtime of a bit below 7 minutes on a M1 Macbook Pro and beat Benjamin Paassen's graph theory approach, which is the fastest pure Python approach that I am aware of (sorry if somebody meanwhile wrote a faster one and I missed it) and served as my benchmark, by approximately a factor of 3 ;-)

The basic idea of my approach: As the runtime of any algorithm for finding all possible combinations of five words with all distinct letters will be $O(n_1*n_2*n_3*n_4*n_5)$, we must keep our numbers of words $n_i$ for the first, second, etc. word as small as only possible. Hence the first thing we must do is to get rid of as many words as possible beforehand and divide our problem into as small as only possible subgroups.

We can also make use of the fact that there are only {{ len(valid.get(0)) }} valid words such as crypt, glyph or nymph in the word list that contain none of the five vowels a, e, i, o, or u. This means that a combination of five words can only contain a word with two vowels if at least one of them contains none. As can be seen easily (almost all of these either contain either an x or a y), the maximum number of words from that vowelless list that can be combined is two.

So we solve the problem in three parts:

* Five words where each contains none or one vowel -- here we can even split the word list by vowel to speed things up: The numbers of max-one-vowel words are {{ {k: len(v) for k, v in vowels.items()} }}.
* One out of {{ len(valid.get(0)) }} words that contains no vowel, three out of {{ len(valid.get(1)) }} words that contain at most one and one out of {{ len(valid.get(2)) }} words that contains at most two vowels
* Two out of {{ len(valid.get(0)) }} words that contain no vowel, one out of {{ len(valid.get(1)) }} words that contains at most one and two out of {{ len(valid.get(3)) }} words that contain at most three vowels

and build everything together to also find {{ len(result) }} combinations as Benjamin did.

## Final thoughts

Even though this approach clearly is no speed champ, my hope is that this *vowel based reduction of possible word combinations that need to be checked* can maybe help somebody else's super fast algorithm to even save a bit more time ;-)

What I found a surprise in the progress is that only {{ len(zero) }} out of these {{ len(result) }}  combinations contain none of those {{ len(valid.get(0)) }} non-vowel words. Originally I thought that combinations with non-vowel words would be the exception rather than the rule, but I had to learn that using one of them opens the chance for another one to contain one word with two vowels, which are {{ round(len(valid.get(2))/len(valid.get(1)),1) }} times more frequent than words with only one vowel.

## The result
Here are the word combinations:


```python
print(len(result))
print(result)
```

    831
    {'brahm,fldxt,pungs,vejoz,wicky', 'chowk,fldxt,jambe,supvr,zingy', 'bhang,fldxt,rumpy,swick,vejoz', 'fldxt,quack,verby,whomp,zings', 'chirp,fldxt,gumby,swank,vejoz', 'bumpy,flang,hdqrs,twick,vejoz', 'fldxt,gconv,jerky,sabzi,whump', 'fldxt,gumby,snick,vejoz,wharp', 'bungy,fldxt,pashm,vejoz,wrick', 'becks,fldxt,ginzo,jarvy,whump', 'backy,fldxt,rings,vejoz,whump', 'busky,chawn,fldxt,grimp,vejoz', 'bumph,cawky,fldxt,grins,vejoz', 'chawk,fldxt,gimps,runby,vejoz', 'brave,fldxt,jocks,whump,zingy', 'bizen,fldxt,gucks,jarvy,whomp', 'bumps,fldxt,hacky,vejoz,wring', 'brick,fldxt,mungy,vejoz,whaps', 'fldxt,gawby,runch,skimp,vejoz', 'fldxt,gumby,spink,vejoz,warch', 'bingy,fldxt,rucks,vejoz,whamp', 'bunks,fldxt,gimpy,vejoz,warch', 'brock,japyx,seqwl,vingt,zhmud', 'bumpy,fldxt,gnash,vejoz,wrick', 'fldxt,grabs,nicky,vejoz,whump', 'crisp,fldxt,gumby,vejoz,whank', 'fldxt,grovy,jumps,whack,zineb', 'flong,japyx,twick,verbs,zhmud', 'chump,fldxt,kirby,swang,vejoz', 'fcomp,gunky,hdqrs,vibex,waltz', 'bumph,fldxt,girny,swack,vejoz', 'bring,fldxt,mucky,pshaw,vejoz', 'benjy,chowk,fldxt,gramp,squiz', 'brugh,campy,fldxt,vejoz,winks', 'becks,fldxt,rajiv,whump,zygon', 'bring,fldxt,sumph,vejoz,wacky', 'bucky,fldxt,pharm,swing,vejoz', 'bumpy,fldxt,hacks,vejoz,wring', 'bumpy,fldxt,shack,vejoz,wring', 'bunks,fldxt,gyric,vejoz,whamp', 'chivw,expdt,flamb,grosz,junky', 'chimp,fldxt,jarvy,snowk,uzbeg', 'bungy,chawk,fldxt,prims,vejoz', 'bumph,fldxt,grosz,njave,wicky', 'bumps,chark,fldxt,vejoz,wingy', 'bumph,fldxt,racks,vejoz,wingy', 'bucky,fldxt,prims,vejoz,whang', 'champ,fldxt,rugby,vejoz,winks', 'bumph,fldxt,girny,vejoz,wacks', 'bungs,fldxt,phyma,vejoz,wrick', 'backy,fldxt,humps,vejoz,wring', 'chump,fldxt,gnaws,kirby,vejoz', 'fldxt,jambe,qophs,vicky,wrung', 'fldxt,quack,verbs,whomp,zingy', 'bungy,chirp,fldxt,mawks,vejoz', 'bumph,fldxt,jocks,warve,zingy', 'bumps,fldxt,ricky,vejoz,whang', 'bumph,crink,fldxt,gawsy,vejoz', 'ambry,fldxt,pungs,vejoz,whick', 'brigs,fldxt,munch,pawky,vejoz', 'bumps,chivw,fldxt,gazon,jerky', 'bungy,fldxt,prick,shawm,vejoz', 'fldxt,nymph,squab,vejoz,wrick', 'chimp,fldxt,grubs,vejoz,wanky', 'bucks,fldxt,hying,vejoz,wramp', 'fldxt,jambe,supvr,whick,zygon', 'brack,fldxt,humpy,vejoz,wings', 'crink,fldxt,gumby,pshaw,vejoz', 'bumpy,carks,fldxt,vejoz,whing', 'bangy,fldxt,sumph,vejoz,wrick', 'chivw,fldxt,grump,jazey,knobs', 'bucky,fldxt,prahm,swing,vejoz', 'backy,fldxt,rumps,vejoz,whing', 'chung,fldxt,kirby,swamp,vejoz', 'crumb,fldxt,kaphs,vejoz,wingy', 'bhang,fldxt,rumpy,vejoz,wicks', 'bonks,chivw,fldxt,graze,jumpy', 'chimb,fldxt,kyung,vejoz,warps', 'fldxt,gryph,njave,quick,wombs', 'bring,fldxt,humpy,vejoz,wacks', 'briny,chump,fldxt,gawks,vejoz', 'burez,chowk,fldxt,jimpy,vangs', 'brake,chivw,fldxt,jumps,zygon', 'fldxt,grimp,quawk,synch,vejoz', 'chimp,fldxt,gowns,jarvy,uzbek', 'bumph,fldxt,rings,vejoz,wacky', 'burps,fldxt,mckay,vejoz,whing', 'fldxt,gconv,jimpy,uzbak,wersh', 'brags,fldxt,nicky,vejoz,whump', 'bucks,fldxt,ringy,vejoz,whamp', 'cheng,fldxt,jimpy,uzbak,vrows', 'fldxt,rugby,snick,vejoz,whamp', 'breck,flows,japyx,vingt,zhmud', 'bumpy,chawk,fldxt,rings,vejoz', 'brack,fldxt,humpy,swing,vejoz', 'chivw,fldxt,jumba,perks,zygon', 'chivw,fldxt,jumby,karez,spong', 'churn,fldxt,gawby,skimp,vejoz', 'fldxt,gconv,jizya,kerbs,whump', 'bungs,chirm,fldxt,pawky,vejoz', 'bucky,fldxt,pharm,vejoz,wings', 'bumps,ching,fldxt,rawky,vejoz', 'fldxt,graph,numbs,vejoz,wicky', 'fldxt,jumpy,verbs,whick,zogan', 'ambry,fldxt,pucks,vejoz,whing', 'bumph,fldxt,gynic,vejoz,warks', 'bungs,fldxt,micky,vejoz,wharp', 'chowk,fldxt,jumpy,verbs,zigan', 'chimb,fldxt,parky,swung,vejoz', 'fldxt,johns,uzbeg,vicky,wramp', 'fldxt,rhumb,spack,vejoz,wingy', 'bumpy,fldxt,shang,vejoz,wrick', 'brusk,chawn,fldxt,gimpy,vejoz', 'burns,chimp,fldxt,gawky,vejoz', 'crawm,fldxt,qophs,uzbek,vying', 'fldxt,ginzo,jumpy,verbs,whack', 'chirk,fldxt,gumby,pawns,vejoz', 'blows,freck,japyx,vingt,zhmud', 'bangy,crump,fldxt,vejoz,whisk', 'bumpy,fldxt,sangh,vejoz,wrick', 'fldxt,grown,jimpy,schav,uzbek', 'brahm,fldxt,picky,swung,vejoz', 'bingy,chump,fldxt,vejoz,warks', 'bucky,fldxt,grimp,shawn,vejoz', 'bingy,fldxt,rumps,vejoz,whack', 'bumph,fldxt,ricky,swang,vejoz', 'chivw,expdt,jumby,klong,zarfs', 'bungy,chimp,fldxt,vejoz,warks', 'chivw,fldxt,grype,junks,zambo', 'bumph,fldxt,rangy,swick,vejoz', 'burny,chimp,fldxt,gawks,vejoz', 'brusk,chawn,fldxt,pigmy,vejoz', 'breva,fldxt,jumps,whick,zygon', 'brack,fldxt,mungy,vejoz,whips', 'blitz,fconv,gryph,judex,mawks', 'birch,fldxt,gunky,swamp,vejoz', 'bumph,carks,fldxt,vejoz,wingy', 'bumps,fldxt,gyric,vejoz,whank', 'braws,chimp,fldxt,gunky,vejoz', 'benjy,fldxt,grosz,quick,whamp', 'bingy,fldxt,mucks,vejoz,wharp', 'chimb,fldxt,pungy,vejoz,warks', 'fldxt,ganev,jumby,qophs,wrick', 'braky,chimp,fldxt,swung,vejoz', 'glack,hdqrs,jowpy,muntz,vibex', 'fldxt,gconv,jumby,karez,whisp', 'cribs,fldxt,kyung,vejoz,whamp', 'fldxt,grapy,numbs,vejoz,whick', 'brick,fldxt,nymph,quags,vejoz', 'breck,fldxt,jowpy,nizam,vughs', 'fldxt,pigmy,scrub,vejoz,whank', 'barmy,fldxt,pungs,vejoz,whick', 'birch,fldxt,mawky,spung,vejoz', 'birky,chung,fldxt,swamp,vejoz', 'fldxt,grabs,nymph,quick,vejoz', 'brawn,fldxt,gimpy,hucks,vejoz', 'bring,fldxt,humpy,swack,vejoz', 'chubs,fldxt,grimp,vejoz,wanky', 'brack,fldxt,jowpy,mizen,vughs', 'fldxt,jarvy,snick,uzbeg,whomp', 'benzo,fldxt,jimpy,vughs,wrack', 'braws,fldxt,gucki,nymph,vejoz', 'bryum,fldxt,pings,vejoz,whack', 'bonks,chivw,fldxt,gazer,jumpy', 'bumpy,fldxt,ricks,vejoz,whang', 'bunch,fldxt,gawky,prims,vejoz', 'bevor,chawk,fldxt,jumpy,zings', 'bumps,fldxt,karch,vejoz,wingy', 'bizen,fldxt,grovy,jumps,whack', 'fldxt,gawby,mirks,punch,vejoz', 'burns,chawk,fldxt,pigmy,vejoz', 'chirk,fldxt,gawby,numps,vejoz', 'brink,chump,fldxt,gawsy,vejoz', 'bumph,casky,fldxt,vejoz,wring', 'bocks,flegm,japyx,thruv,windz', 'burga,fldxt,nymph,vejoz,wicks', 'expdt,furzy,gconv,jambs,whilk', 'fldxt,gconv,herbs,jimpy,quawk', 'bumpy,chark,fldxt,vejoz,wings', 'brevi,fldxt,jumps,whack,zygon', 'bumps,chivw,fldxt,jorge,knyaz', 'bucky,fldxt,singh,vejoz,wramp', 'baker,chivw,fldxt,jumps,zygon', 'behav,fldxt,jumps,wrick,zygon', 'chivw,expdt,furzy,jambs,klong', 'fldxt,gumby,prink,schwa,vejoz', 'bumph,fldxt,grosz,jacky,vinew', 'bumpy,ching,fldxt,vejoz,warks', 'fldxt,jumba,qophs,vying,wreck', 'chums,fldxt,gawby,prink,vejoz', 'bumpy,fldxt,grins,vejoz,whack', 'bumps,chawk,fldxt,ringy,vejoz', 'bring,casky,fldxt,vejoz,whump', 'ampyx,bejig,fconv,hdqrs,klutz', 'benjy,chimp,fldxt,grosz,quawk', 'chivw,expdt,flank,grosz,jumby', 'bumph,fldxt,gowks,javer,zincy', 'braws,fldxt,nymph,quick,vejoz', 'bumph,fldxt,gawky,scrin,vejoz', 'bumph,fldxt,jives,wrack,zygon', 'chunk,fldxt,gawby,prims,vejoz', 'fldxt,gibus,nymph,vejoz,wrack', 'crimp,fldxt,gawby,hunks,vejoz', 'bungy,fldxt,phasm,vejoz,wrick', 'fldxt,gconv,jerky,squib,whamp', 'brusk,campy,fldxt,vejoz,whing', 'brick,fldxt,pungy,shawm,vejoz', 'bucky,fldxt,prahm,vejoz,wings', 'bumpy,fldxt,ginks,vejoz,warch', 'fldxt,gryph,minbu,swack,vejoz', 'chank,fldxt,jimpy,uzbeg,vrows', 'chirp,fldxt,gawky,numbs,vejoz', 'fldxt,gryph,manqu,swick,vejoz', 'curbs,fldxt,pigmy,vejoz,whank', 'fldxt,jumby,qophs,vegan,wrick', 'bring,cawky,fldxt,humps,vejoz', 'bungy,fldxt,prahm,vejoz,wicks', 'fldxt,gawby,murph,nicks,vejoz', 'bocks,fldxt,javer,whump,zingy', 'bumpy,fldxt,karch,vejoz,wings', 'chimp,fldxt,gawks,runby,vejoz', 'crump,fldxt,gawby,knish,vejoz', 'brawn,fldxt,pigmy,shuck,vejoz', 'bumpy,chivw,fldxt,gazon,jerks', 'bingy,chawk,fldxt,rumps,vejoz', 'backy,fldxt,grins,vejoz,whump', 'blitz,fconv,gawky,hdqrs,pumex', 'bumpy,chawk,fldxt,girns,vejoz', 'becky,fldxt,jumps,voraz,whing', 'breva,fldxt,jocks,whump,zingy', 'bucky,fldxt,prism,vejoz,whang', 'bargh,fldxt,numps,vejoz,wicky', 'brick,fldxt,gnaws,humpy,vejoz', 'braze,fldxt,jocks,vying,whump', 'fldxt,jacko,verby,whump,zings', 'borgh,fldxt,jumps,vicky,wanze', 'braws,chimp,fldxt,kyung,vejoz', 'frock,japyx,seqwl,vingt,zhmud', 'bumph,fldxt,jacks,wiver,zygon', 'brick,fldxt,spumy,vejoz,whang', 'backs,fldxt,ringy,vejoz,whump', 'becky,fldxt,jumps,vizor,whang', 'bumph,fldxt,ginzo,jacks,wyver', 'bumph,cawky,fldxt,girns,vejoz', 'bumps,chyak,fldxt,vejoz,wring', 'brack,fldxt,jowpy,vughs,zemni', 'briny,fldxt,gucks,vejoz,whamp', 'chawk,fldxt,ginzo,jumpy,verbs', 'brugh,fldxt,micky,pawns,vejoz', 'avick,benjy,fldxt,grosz,whump', 'bowls,freck,japyx,vingt,zhmud', 'fldxt,griph,numbs,vejoz,wacky', 'bumph,fldxt,gnaws,ricky,vejoz', 'breva,chowk,fldxt,jumps,zingy', 'bingy,fldxt,humps,vejoz,wrack', 'bumph,fldxt,gansy,vejoz,wrick', 'chawn,fldxt,gumby,skirp,vejoz', 'bunch,fldxt,pigmy,vejoz,warks', 'brack,fldxt,mungy,vejoz,whisp', 'fldxt,gumby,prick,shawn,vejoz', 'fldxt,jimpy,schav,uzbek,wrong', 'burgh,fldxt,micky,pawns,vejoz', 'fldxt,kinch,rugby,swamp,vejoz', 'expdt,gconv,jumby,whilk,zarfs', 'bungs,chimp,fldxt,rawky,vejoz', 'bunch,fldxt,grips,mawky,vejoz', 'fldxt,gucks,jarvy,whomp,zineb', 'fldxt,gawks,nymph,urbic,vejoz', 'block,fremt,japyx,vughs,windz', 'fldxt,gumby,parch,vejoz,winks', 'chivw,fldxt,jumpy,kerbs,zogan', 'fldxt,jarvy,mowch,pinks,uzbeg', 'bucks,fldxt,prahm,vejoz,wingy', 'brick,fldxt,pamhy,swung,vejoz', 'flock,hdqrs,jumpy,twang,vibex', 'ampyx,bortz,chivw,fjeld,gunks', 'fjord,glyph,muntz,vibex,wacks', 'burps,fldxt,micky,vejoz,whang', 'burps,fldxt,mangy,vejoz,whick', 'chimb,fldxt,gunky,vejoz,wraps', 'flamb,hdqrs,pungy,twick,vejoz', 'fldxt,gimpy,scrub,vejoz,whank', 'crink,fldxt,gumby,vejoz,whaps', 'bumps,fldxt,hying,vejoz,wrack', 'burny,fldxt,gimps,vejoz,whack', 'churm,fldxt,gawby,pinks,vejoz', 'bizen,chawk,fldxt,grovy,jumps', 'chump,fldxt,gawby,rinks,vejoz', 'chomp,fldxt,jarvy,uzbeg,winks', 'backy,fldxt,murph,swing,vejoz', 'fldxt,gawby,munch,skirp,vejoz', 'fldxt,grubs,mawky,pinch,vejoz', 'expwy,flack,hdqrs,jumbo,vingt', 'chomp,fldxt,jarvy,swing,uzbek', 'bring,cawky,fldxt,sumph,vejoz', 'angry,bumps,fldxt,vejoz,whick', 'chivw,fldxt,gazer,jumpy,knobs', 'bryum,fldxt,gawks,pinch,vejoz', 'bungs,fldxt,mawky,prich,vejoz', 'backy,fldxt,grump,vejoz,whins', 'braky,chump,fldxt,vejoz,wings', 'becks,fultz,japyx,mordv,whing', 'bungy,fldxt,ricks,vejoz,whamp', 'bring,fldxt,mucky,vejoz,whaps', 'fldxt,gripy,numbs,vejoz,whack', 'brack,fldxt,spumy,vejoz,whing', 'fldxt,gravy,jocks,whump,zineb', 'fldxt,jumby,pecks,voraz,whing', 'bungy,fldxt,prick,vejoz,whams', 'bortz,chivw,dunks,flegm,japyx', 'chowk,fldxt,jumby,prize,vangs', 'fjord,glyph,muntz,swack,vibex', 'bumph,fldxt,vejoz,wrick,yangs', 'chivw,fldxt,gazon,jumpy,krebs', 'bingy,fldxt,shuck,vejoz,wramp', 'bangy,fldxt,murph,swick,vejoz', 'bumpy,chirk,fldxt,gnaws,vejoz', 'bungy,fldxt,pharm,swick,vejoz', 'bumps,chivw,fldxt,jerky,zogan', 'chirm,fldxt,gawby,punks,vejoz', 'chimb,fldxt,pungs,rawky,vejoz', 'bezan,fldxt,grovy,jumps,whick', 'fldxt,garbs,nicky,vejoz,whump', 'chivw,fldxt,graze,jumby,knosp', 'chirm,fldxt,gawby,spunk,vejoz', 'fldxt,jumby,navig,qophs,wreck', 'brave,chowk,fldxt,jumpy,zings', 'barmy,fldxt,spung,vejoz,whick', 'bring,chawk,fldxt,spumy,vejoz', 'bumph,fldxt,jacks,vower,zingy', 'bongs,chivw,fldxt,jumpy,karez', 'brick,fldxt,pungy,vejoz,whams', 'fldxt,gumby,pirns,vejoz,whack', 'fldxt,gconv,jumpy,whisk,zebra', 'bryum,fldxt,picks,vejoz,whang', 'bring,fldxt,humps,vejoz,wacky', 'bucky,fldxt,nighs,vejoz,wramp', 'cawky,fldxt,griph,numbs,vejoz', 'bumpy,chivw,fldxt,jerks,zogan', 'fldxt,gumby,pinks,vejoz,warch', 'becks,fldxt,jumpy,voraz,whing', 'birch,fldxt,kyung,swamp,vejoz', 'angry,bumph,fldxt,swick,vejoz', 'chirk,fldxt,gumby,spawn,vejoz', 'benjy,fldxt,oghuz,vamps,wrick', 'crumb,fldxt,pawky,singh,vejoz', 'chowk,fldxt,gravy,jumps,zineb', 'fldxt,garbs,nymph,quick,vejoz', 'fldxt,jacko,verbs,whump,zingy', 'benjy,fldxt,gizmo,supvr,whack', 'burgh,fldxt,nicky,swamp,vejoz', 'bigam,expwy,fconv,hdqrs,klutz', 'bangy,fldxt,rumps,vejoz,whick', 'fldxt,gazon,jumps,verby,whick', 'bejig,fldxt,knyaz,mowch,supvr', 'fldxt,ganch,jimpy,uzbek,vrows', 'bucks,fldxt,javer,whomp,zingy', 'chivw,fjord,klutz,mangy,pbxes', 'brugh,campy,fldxt,swink,vejoz', 'bingy,fldxt,sumph,vejoz,wrack', 'chowk,fldxt,jumby,paver,zings', 'chank,fldxt,jowpy,mirvs,uzbeg', 'fldxt,ginzo,jacks,verby,whump', 'barky,chump,fldxt,swing,vejoz', 'chimb,fldxt,sprug,vejoz,wanky', 'fldxt,gawby,jumps,kench,vizor', 'busky,cramp,fldxt,vejoz,whing', 'burez,fldxt,jacks,vying,whomp', 'crwth,gambs,kylix,pfund,vejoz', 'bucks,fldxt,gramp,vejoz,whiny', 'brims,chung,fldxt,pawky,vejoz', 'brugh,fldxt,micky,spawn,vejoz', 'busky,crimp,fldxt,vejoz,whang', 'dwarf,glyph,jocks,muntz,vibex', 'baken,chivw,fldxt,grosz,jumpy', 'chivw,fldxt,jerks,pungy,zambo', 'chump,fldxt,jakob,wyver,zings', 'bumph,crang,fldxt,skiwy,vejoz', 'chivw,fldxt,jerky,spung,zambo', 'burgs,chimp,fldxt,vejoz,wanky', 'dhikr,expwy,fultz,gconv,jambs', 'brock,flews,japyx,vingt,zhmud', 'brack,fldxt,jives,whump,zygon', 'bumph,fldxt,gracy,swink,vejoz', 'bewig,fconv,hdqrs,japyx,klutz', 'backy,fldxt,murph,vejoz,wings', 'chowk,fldxt,jumps,verby,zigan', 'chimb,fldxt,gunky,vejoz,warps', 'bucks,fldxt,primy,vejoz,whang', 'chink,fldxt,gumby,vejoz,warps', 'chivw,enzym,fldxt,jakob,sprug', 'burns,fldxt,pigmy,vejoz,whack', 'exptl,gconv,hdqrs,jumby,wakif', 'braze,fldxt,gconv,jumpy,whisk', 'bunks,fldxt,pigmy,vejoz,warch', 'bucks,fldxt,mingy,vejoz,wharp', 'bunch,fldxt,gawks,primy,vejoz', 'bevor,fldxt,jumps,whack,zingy', 'brick,fldxt,mungy,pshaw,vejoz', 'chowk,fldxt,jumps,vying,zebra', 'bumph,fldxt,rangy,vejoz,wicks', 'crumb,fldxt,kyang,vejoz,whisp', 'bichy,fldxt,grump,swank,vejoz', 'chivw,fldxt,gazer,jumby,knops', 'fldxt,gumby,parks,vejoz,winch', 'fldxt,ginzo,jacky,verbs,whump', 'brags,fldxt,nymph,quick,vejoz', 'brigs,fldxt,nymph,quack,vejoz', 'ampyx,crwth,fdubs,kling,vejoz', 'braws,chunk,fldxt,gimpy,vejoz', 'benjy,fldxt,gucks,vizor,whamp', 'bryum,fldxt,spang,vejoz,whick', 'ampyx,crwth,fdubs,glink,vejoz', 'barky,chump,fldxt,vejoz,wings', 'chang,fldxt,jimpy,uzbek,vrows', 'crink,fldxt,gawby,humps,vejoz', 'bumph,fldxt,ringy,swack,vejoz', 'brack,fldxt,pungy,vejoz,whims', 'burgs,fldxt,mawky,pinch,vejoz', 'bizen,fldxt,grovy,jacks,whump', 'fldxt,jarvy,mowch,spink,uzbeg', 'bumpy,fldxt,singh,vejoz,wrack', 'bucky,fldxt,prams,vejoz,whing', 'carby,fldxt,ginks,vejoz,whump', 'frack,jowly,pbxes,vingt,zhmud', 'chimp,fldxt,jarvy,swonk,uzbeg', 'bangy,fldxt,humps,vejoz,wrick', 'burgh,cawny,fldxt,skimp,vejoz', 'fldxt,gimps,runby,vejoz,whack', 'backs,fldxt,girny,vejoz,whump', 'bring,fldxt,spumy,vejoz,whack', 'fldxt,gumby,prich,swank,vejoz', 'birch,fldxt,gawky,numps,vejoz', 'brawn,fldxt,hucks,pigmy,vejoz', 'flack,hdqrs,jowpy,muntz,vibex', 'birky,chump,fldxt,swang,vejoz', 'brave,fldxt,jumps,whick,zygon', 'bucks,fldxt,pamhy,vejoz,wring', 'bumps,chowk,fldxt,javer,zingy', 'fldxt,gazon,jumpy,verbs,whick', 'curby,fldxt,gimps,vejoz,whank', 'fldxt,gconv,hawks,jumby,prize', 'chivw,fldxt,gazer,jumby,knosp', 'frowl,jacky,pbxes,vingt,zhmud', 'bangy,fldxt,ricks,vejoz,whump', 'bhang,fldxt,spumy,vejoz,wrick', 'birky,champ,fldxt,swung,vejoz', 'busky,chimp,fldxt,vejoz,wrang', 'crips,fldxt,gumby,vejoz,whank', 'chivw,fldxt,jumps,kebar,zygon', 'chirm,fldxt,jowpy,uzbek,vangs', 'brick,fldxt,humpy,swang,vejoz', 'chink,fldxt,gawby,rumps,vejoz', 'bunch,fldxt,mawky,prigs,vejoz', 'fcomp,hdqrs,junky,vibex,waltz', 'chink,fldxt,gumby,vejoz,wraps', 'chwas,fldxt,gumby,prink,vejoz', 'bumpy,fldxt,racks,vejoz,whing', 'bucks,fldxt,phyma,vejoz,wring', 'banks,fldxt,gyric,vejoz,whump', 'braze,chowk,fldxt,jumps,vying', 'busky,champ,fldxt,vejoz,wring', 'crank,fldxt,gumby,vejoz,whips', 'becks,frowl,japyx,vingt,zhmud', 'bungs,fldxt,primy,vejoz,whack', 'fldxt,gconv,jizya,krebs,whump', 'bumph,fldxt,gravy,jocks,wizen', 'fldxt,gumby,nicks,vejoz,wharp', 'fldxt,jumps,verby,whick,zogan', 'bungy,chawk,fldxt,prism,vejoz', 'birch,fldxt,mawks,pungy,vejoz', 'brims,fldxt,gawky,punch,vejoz', 'brack,fldxt,sumph,vejoz,wingy', 'cheng,fldxt,jowpy,mirvs,uzbak', 'brick,fldxt,swung,vejoz,yamph', 'bevor,fldxt,jacky,whump,zings', 'burny,fldxt,gamps,vejoz,whick', 'burps,chawk,fldxt,mingy,vejoz', 'birks,fldxt,gconv,jazey,whump', 'chomp,fldxt,jarvy,swink,uzbeg', 'brisk,cangy,fldxt,vejoz,whump', 'bunch,fldxt,gramp,skiwy,vejoz', 'bowge,flick,hdqrs,japyx,muntz', 'chimp,fldxt,quawk,verbs,zygon', 'bilks,fconv,grewt,japyx,zhmud', 'fcomp,hdqrs,kyung,vibex,waltz', 'bumph,fldxt,vejoz,wring,yacks', 'busky,ching,fldxt,vejoz,wramp', 'fldxt,quick,verbs,whamp,zygon', 'bangs,fldxt,rumpy,vejoz,whick', 'bumph,fldxt,gyric,swank,vejoz', 'cribs,fldxt,gunky,vejoz,whamp', 'fldxt,gimps,jarvy,nowch,uzbek', 'bench,fldxt,gawky,jumps,vizor', 'birks,cangy,fldxt,vejoz,whump', 'benzo,fldxt,gravy,jumps,whick', 'cribs,fldxt,nymph,quawk,vejoz', 'burgh,fldxt,micky,spawn,vejoz', 'fldxt,gnaws,picky,rhumb,vejoz', 'fldxt,pings,rhumb,vejoz,wacky', 'fldxt,gumby,njave,qophs,wrick', 'braky,chump,fldxt,swing,vejoz', 'bhang,crump,fldxt,skiwy,vejoz', 'fldxt,pangs,rhumb,vejoz,wicky', 'bungs,fldxt,prahm,vejoz,wicky', 'bungs,fldxt,pharm,vejoz,wicky', 'barks,fldxt,gynic,vejoz,whump', 'bonze,fldxt,jimpy,vughs,wrack', 'backs,fldxt,grump,vejoz,whiny', 'bumpy,fldxt,grosz,njave,whick', 'expwy,flock,hdqrs,jumba,vingt', 'bruja,chivw,fldxt,kemps,zygon', 'bryum,fldxt,pangs,vejoz,whick', 'fldxt,gconv,jerky,squiz,whamp', 'cribs,fldxt,kyang,vejoz,whump', 'bumph,fldxt,jacko,wyver,zings', 'backy,fldxt,sumph,vejoz,wring', 'fldxt,jumba,qophs,wreck,zingy', 'fldxt,jumby,qophs,wreck,zigan', 'fldxt,gryph,minbu,vejoz,wacks', 'bumph,fldxt,gconv,jerky,swazi', 'bunch,fldxt,gimpy,vejoz,warks', 'fldxt,jumby,pecks,vizor,whang', 'burps,fldxt,mingy,vejoz,whack', 'chaws,fldxt,gumby,prink,vejoz', 'bongs,chivw,fremd,japyx,klutz', 'chimp,fldxt,jarvy,knows,uzbeg', 'barks,chump,fldxt,vejoz,wingy', 'becks,fldxt,jumpy,vizor,whang', 'bumps,fldxt,jahve,wrick,zygon', 'bevor,fldxt,jumpy,whack,zings', 'fultz,jacky,mordv,pbxes,whing', 'burga,fldxt,nymph,swick,vejoz', 'chivw,fldxt,grosz,jambe,punky', 'brows,fleck,japyx,vingt,zhmud', 'chivw,fldxt,graze,jumpy,knobs', 'birky,chump,fldxt,gnaws,vejoz', 'chump,fldxt,gowks,jarvy,zineb', 'crank,fldxt,gumby,vejoz,whisp', 'bungy,fldxt,prahm,swick,vejoz', 'bucks,fldxt,girny,vejoz,whamp', 'evang,fldxt,jumby,qophs,wrick', 'fldxt,gconv,jimpy,shrew,uzbak', 'cawky,fldxt,pings,rhumb,vejoz', 'chomp,fldxt,quawk,verby,zings', 'banky,crump,fldxt,vejoz,whigs', 'brigs,fldxt,nymph,quawk,vejoz', 'bench,fldxt,gawks,jumpy,vizor', 'flowk,hdqrs,imcnt,japyx,uzbeg', 'brims,chawk,fldxt,pungy,vejoz', 'fangy,hdqrs,plumb,twick,vejoz', 'bumps,fldxt,rangy,vejoz,whick', 'fldxt,nymph,quags,vejoz,wrick', 'backy,fldxt,girns,vejoz,whump', 'benjy,chawk,fldxt,gizmo,supvr', 'bumph,fldxt,jacky,vower,zings', 'comps,fldxt,jarvy,uzbek,whing', 'fldxt,gawby,murks,pinch,vejoz', 'backs,fldxt,humpy,vejoz,wring', 'bangs,fldxt,murph,vejoz,wicky', 'brogh,fldxt,jumps,vicky,wanze', 'brack,fldxt,humps,vejoz,wingy', 'chang,fldxt,jowpy,mirvs,uzbek', 'bewig,flock,hdqrs,japyx,muntz', 'bungy,fldxt,prams,vejoz,whick', 'fldxt,gumby,prawn,shick,vejoz', 'brusk,champ,fldxt,vejoz,wingy', 'bench,fldxt,grosz,jimpy,quawk', 'bungs,fldxt,ricky,vejoz,whamp', 'bumph,fldxt,javer,wicks,zygon', 'bring,fldxt,vejoz,whump,yacks', 'chawn,fldxt,rugby,skimp,vejoz', 'bungy,crimp,fldxt,hawks,vejoz', 'barms,fldxt,pungy,vejoz,whick', 'bumpy,crank,fldxt,vejoz,whigs', 'bumph,fldxt,jacks,wrive,zygon', 'chivw,fldxt,graze,jumby,knops', 'chivw,fldxt,gerbo,jumps,knyaz', 'burez,fldxt,gconv,hawks,jimpy', 'bungy,fldxt,skimp,vejoz,warch', 'bungs,crimp,fldxt,hawky,vejoz', 'fldxt,jerks,vocab,whump,zingy', 'bumpy,fldxt,hicks,vejoz,wrang', 'chowk,fldxt,jumby,verpa,zings', 'chivw,fldxt,grosz,jumby,knape', 'bejig,chump,fldxt,knyaz,vrows', 'chawk,fldxt,grovy,jumps,zineb', 'brusk,fldxt,gynic,vejoz,whamp', 'bucky,fldxt,gramp,vejoz,whins', 'bichy,fldxt,gunks,vejoz,wramp', 'bucks,fldxt,vejoz,wring,yamph', 'bunch,fldxt,mawky,sprig,vejoz', 'burns,fldxt,gimpy,vejoz,whack', 'bungy,fldxt,prism,vejoz,whack', 'fcomp,gawby,hdqrs,klutz,vixen', 'bovld,freck,japyx,muntz,whigs', 'busky,fldxt,gramp,vejoz,winch', 'brick,fldxt,phyma,swung,vejoz', 'bemix,fultz,gconv,hdqrs,pawky', 'chawk,fldxt,gripy,numbs,vejoz', 'bunch,fldxt,gawky,prism,vejoz', 'bumpy,fldxt,hangs,vejoz,wrick', 'bucky,fldxt,phasm,vejoz,wring', 'bungs,fldxt,vejoz,wrick,yamph', 'bryum,fldxt,packs,vejoz,whing', 'birny,fldxt,gucks,vejoz,whamp', 'chivw,fjord,glaky,muntz,pbxes', 'carvy,fldxt,jinks,uzbeg,whomp', 'chivw,fldxt,jerky,pungs,zambo', 'brugh,fldxt,nicky,swamp,vejoz', 'bumpy,fldxt,shick,vejoz,wrang', 'bejig,fldxt,nymph,quack,vrows', 'chimb,expwy,fjord,klutz,vangs', 'bangy,flump,hdqrs,twick,vejoz', 'chump,fldxt,gawby,kirns,vejoz', 'burps,ching,fldxt,mawky,vejoz', 'fldxt,picky,rhumb,swang,vejoz', 'chimb,fldxt,gawky,snurp,vejoz', 'crumb,fldxt,pisky,vejoz,whang', 'fldxt,jerky,vocab,whump,zings', 'brick,fldxt,vejoz,whump,yangs', 'bungy,fldxt,hicks,vejoz,wramp', 'crumb,fldxt,gipsy,vejoz,whank', 'bawke,fldxt,gconv,jimpy,qursh', 'backs,fldxt,murph,vejoz,wingy', 'bungy,fldxt,ramps,vejoz,whick', 'bungs,chawk,fldxt,primy,vejoz', 'curby,fldxt,ginks,vejoz,whamp', 'fldxt,gawby,punch,smirk,vejoz', 'chimb,fldxt,gawky,spurn,vejoz', 'bring,chums,fldxt,pawky,vejoz', 'bucky,fldxt,ramps,vejoz,whing', 'bryum,fldxt,spack,vejoz,whing', 'bangs,fldxt,ricky,vejoz,whump', 'bizen,chump,fldxt,gowks,jarvy', 'fldxt,packs,rhumb,vejoz,wingy', 'crumb,fldxt,kyang,vejoz,whips', 'bonze,fldxt,gravy,jumps,whick', 'curbs,fldxt,gimpy,vejoz,whank', 'brick,fldxt,nymph,squaw,vejoz', 'bumpy,chowk,fldxt,javer,zings', 'brahm,fldxt,spung,vejoz,wicky', 'flong,jarvy,pbxes,twick,zhmud', 'backs,fldxt,rumpy,vejoz,whing', 'bungy,fldxt,sparm,vejoz,whick', 'chink,fldxt,rugby,swamp,vejoz', 'bumph,fldxt,javer,swick,zygon', 'fldxt,ganch,jowpy,mirvs,uzbek', 'fldxt,gawky,numbs,prich,vejoz', 'bumps,fldxt,hicky,vejoz,wrang', 'burgh,campy,fldxt,swink,vejoz', 'chimp,fldxt,gawby,knurs,vejoz', 'bangs,fldxt,humpy,vejoz,wrick', 'bumpy,crang,fldxt,vejoz,whisk', 'bucky,fldxt,rings,vejoz,whamp', 'fldxt,gumby,pinch,vejoz,warks', 'bumpy,chirk,fldxt,swang,vejoz', 'bungs,fldxt,hicky,vejoz,wramp', 'bunch,fldxt,gripy,mawks,vejoz', 'bhang,fldxt,rumps,vejoz,wicky', 'fldxt,gconv,jumby,karez,whips', 'burez,fldxt,jocks,vying,whamp', 'chivw,fldxt,jumpy,krebs,zogan', 'fldxt,gumby,kinch,vejoz,warps', 'chomp,fldxt,quawk,verbs,zingy', 'bleck,frows,japyx,vingt,zhmud', 'bonks,chivw,fldxt,grump,jazey', 'bucky,fldxt,javer,whomp,zings', 'bungy,hdqrs,lampf,twick,vejoz', 'bungy,fldxt,micks,vejoz,wharp', 'bumph,fldxt,grins,vejoz,wacky', 'carby,fldxt,kings,vejoz,whump', 'bucky,fldxt,prang,vejoz,whims', 'fldxt,gumby,scrip,vejoz,whank', 'bunch,fldxt,gimps,rawky,vejoz', 'bevor,fldxt,jacks,whump,zingy', 'fldxt,gamps,runby,vejoz,whick', 'exptl,fconv,gawby,hdqrs,mujik', 'ampyx,flung,hdqrs,twick,vejoz', 'bungs,chirp,fldxt,mawky,vejoz', 'crumb,fldxt,nighs,pawky,vejoz', 'brock,fldxt,jimpy,vughs,wanze', 'brins,chump,fldxt,gawky,vejoz', 'fldxt,kumbi,psych,vejoz,wrang', 'ambry,fldxt,spung,vejoz,whick', 'fldxt,jarvy,nowch,skimp,uzbeg', 'fldxt,rhumb,spang,vejoz,wicky', 'bingy,crump,fldxt,hawks,vejoz', 'chunk,fldxt,gawby,prism,vejoz', 'chivw,fldxt,jambs,puker,zygon', 'fldxt,gawby,murph,snick,vejoz', 'brews,flock,japyx,vingt,zhmud', 'brahm,fldxt,pungy,vejoz,wicks', 'bumph,fldxt,jocks,wyver,zigan', 'bumpy,fldxt,kings,vejoz,warch', 'bumpy,fldxt,gawks,rinch,vejoz', 'brigs,fldxt,mawky,punch,vejoz', 'chawk,fldxt,ginzo,jumps,verby', 'bungy,chirk,fldxt,swamp,vejoz', 'burgh,campy,fldxt,vejoz,winks', 'chimb,fldxt,pawky,rungs,vejoz', 'fldxt,ginzo,jumps,verby,whack', 'fldxt,jumbo,pshav,wreck,zingy', 'breva,chowk,fldxt,jumpy,zings', 'bucky,fldxt,sparm,vejoz,whing', 'burns,chawk,fldxt,gimpy,vejoz', 'brawn,fldxt,kopje,vughs,zymic', 'bejan,fldxt,grosz,vicky,whump', 'fldxt,ghbor,jumps,vicky,wanze', 'bizen,chowk,fldxt,gravy,jumps', 'fldxt,gawby,kinch,rumps,vejoz', 'brawn,fldxt,gimpy,shuck,vejoz', 'bumps,chawk,fldxt,girny,vejoz', 'chivw,fldxt,gazon,jumpy,kerbs', 'curby,fldxt,kings,vejoz,whamp', 'fldxt,gumby,kinch,vejoz,wraps', 'bronk,chivw,fldxt,gazes,jumpy', 'bumph,cawky,fldxt,rings,vejoz', 'bumps,fldxt,javer,whick,zygon', 'bungy,fldxt,prims,vejoz,whack', 'brevi,chawk,fldxt,jumps,zygon', 'chivw,fldxt,gazon,jumby,perks', 'fldxt,grosz,jumby,paven,whick', 'burny,chawk,fldxt,gimps,vejoz', 'fldxt,grubs,nicky,vejoz,whamp', 'fldxt,gumby,parch,swink,vejoz', 'birny,chump,fldxt,gawks,vejoz', 'bangy,fldxt,murph,vejoz,wicks', 'bumph,fldxt,grovy,jacks,wizen', 'brugh,cawny,fldxt,skimp,vejoz', 'bumpy,fldxt,karch,swing,vejoz', 'birch,fldxt,mawky,pungs,vejoz', 'chivw,fldxt,jumby,perks,zogan', 'brigs,chump,fldxt,vejoz,wanky', 'fldxt,nicks,rugby,vejoz,whamp', 'bumps,fldxt,gawky,rinch,vejoz', 'fldxt,nymph,squib,vejoz,wrack', 'bumpy,chawk,fldxt,grins,vejoz', 'bingy,fldxt,racks,vejoz,whump', 'bingy,carks,fldxt,vejoz,whump', 'benjy,chump,fldxt,gawks,vizor', 'bungy,fldxt,pharm,vejoz,wicks', 'chomp,fldxt,jarvy,uzbek,wings', 'bizen,fldxt,gravy,jocks,whump', 'bucky,flimp,hdqrs,twang,vejoz', 'bumpy,chark,fldxt,swing,vejoz', 'braws,chunk,fldxt,pigmy,vejoz', 'chimp,fldxt,rugby,swank,vejoz', 'bumph,fldxt,ringy,vejoz,wacks', 'brevi,fldxt,jacks,whump,zygon', 'fldxt,jumby,prove,whack,zings', 'fjord,gucks,nymph,vibex,waltz', 'breck,japyx,vingt,wolfs,zhmud', 'bumph,fldxt,girns,vejoz,wacky', 'ampyx,bewig,fconv,hdqrs,klutz', 'bumpy,fldxt,rings,vejoz,whack', 'brisk,fldxt,gconv,jazey,whump', 'bungy,fldxt,mawks,prich,vejoz', 'curby,fldxt,skimp,vejoz,whang', 'fldxt,jocks,verby,whump,zigan', 'fldxt,jarvy,nicks,uzbeg,whomp', 'fldxt,gryph,jambs,quick,woven', 'brahm,fldxt,pucks,vejoz,wingy', 'bumps,fldxt,girny,vejoz,whack', 'crink,fldxt,gawby,sumph,vejoz', 'bungy,cramp,fldxt,vejoz,whisk', 'bumpy,fldxt,nighs,vejoz,wrack', 'fldxt,gumby,hicks,prawn,vejoz', 'fldxt,jocks,vying,whump,zebra', 'fldxt,jumby,speck,vizor,whang', 'bring,fldxt,vejoz,whamp,yucks', 'bumph,fldxt,gnars,vejoz,wicky', 'bevor,chawk,fldxt,jumps,zingy', 'angry,bumph,fldxt,vejoz,wicks', 'barky,chimp,fldxt,swung,vejoz', 'bungy,fldxt,shick,vejoz,wramp', 'fldxt,gconv,jerky,saqib,whump', 'bingy,fldxt,murph,swack,vejoz', 'bungs,fldxt,pamhy,vejoz,wrick', 'dumbs,fritz,gconv,japyx,whelk', 'bucky,fldxt,grins,vejoz,whamp', 'brick,fldxt,gansy,vejoz,whump', 'fldxt,grovy,jacks,whump,zineb', 'fldxt,jarvy,mowch,pings,uzbek', 'chawk,fldxt,jumby,prove,zings', 'champ,fldxt,kirby,swung,vejoz', 'bumpy,fldxt,girns,vejoz,whack', 'breck,fowls,japyx,vingt,zhmud', 'bumps,fldxt,ringy,vejoz,whack', 'fldxt,ginzo,jumby,pshav,wreck', 'fldxt,gryph,manqu,vejoz,wicks', 'burgs,fldxt,nicky,vejoz,whamp', 'bumph,fldxt,jocks,waver,zingy', 'bucks,fldxt,pharm,vejoz,wingy', 'bryum,fldxt,spick,vejoz,whang', 'bumpy,fldxt,gnars,vejoz,whick', 'chawk,fldxt,gumby,pirns,vejoz', 'bryum,chawk,fldxt,pings,vejoz', 'fldxt,namby,sprug,vejoz,whick', 'churm,fldxt,gawby,spink,vejoz', 'fldxt,grimp,subch,vejoz,wanky', 'brahm,fldxt,pungy,swick,vejoz', 'breck,fldxt,jowpy,nazim,vughs', 'chimb,fldxt,rawky,spung,vejoz', 'fldxt,gumby,spark,vejoz,winch', 'bumph,fldxt,gracy,vejoz,winks', 'fldxt,jumby,speck,voraz,whing', 'chimb,fldxt,kyung,vejoz,wraps', 'chivw,fldxt,grosz,jumby,pekan', 'brave,chowk,fldxt,jumps,zingy', 'bumpy,flick,hdqrs,twang,vejoz', 'chivw,fldxt,grebo,jumps,knyaz', 'bruja,chivw,fldxt,skemp,zygon', 'bingy,fldxt,murph,vejoz,wacks', 'fldxt,mawks,pinch,rugby,vejoz', 'bingy,fldxt,hucks,vejoz,wramp', 'brims,fldxt,pungy,vejoz,whack', 'bucky,fldxt,girns,vejoz,whamp', 'exptl,gconv,hdqrs,jumby,kafiz', 'chowk,fldxt,jumby,parve,zings', 'crumb,fldxt,spiky,vejoz,whang', 'bawke,fultz,gconv,hdqrs,jimpy', 'barmy,fldxt,pucks,vejoz,whing', 'fldxt,given,jumby,qophs,wrack', 'bring,fldxt,psych,quawk,vejoz', 'bucky,fldxt,pashm,vejoz,wring', 'break,chivw,fldxt,jumps,zygon', 'champ,fldxt,rugby,swink,vejoz', 'bumph,fldxt,gravy,jocks,winze', 'bumph,fldxt,grovy,jacks,winze', 'crumb,fldxt,hawky,pings,vejoz'}

