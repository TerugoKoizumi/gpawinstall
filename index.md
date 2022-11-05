Cuの格子定数を収束させるにあたって

`https://wiki.fysik.dtu.dk/gpaw/tutorialsexercises/structureoptimization/lattice_constants/lattice_constants.html`

このウェブ上に書いてある通り進めるとAlでは上手くグラフの取得までできます。

Cuの場合、計算自体は進むのですが、`analysis.py`を実行すると以下のエラーが出てきます。

<br>

↓カットオフエネルギーを変化させて計算するスクリプト
```
import numpy as np
from ase.build import bulk
from gpaw import GPAW, PW

a0 = 3.61
cu = bulk('Cu', 'fcc', a=a0)
cell0 = cu.cell

for ecut in range(200, 801, 50):
    cu.calc = GPAW(mode=PW(ecut),
                   xc='PBE',
                   kpts=(8, 8, 8),
                   basis='dzp',
                   txt=f'Cu-{ecut}.txt')
    for eps in np.linspace(-0.02, 0.02, 5):
        cu.cell = (1 + eps) * cell0
        cu.get_potential_energy()

```

↓`analysis.py`
```
import matplotlib.pyplot as plt

from ase.build import bulk
from ase.eos import EquationOfState
from ase.io import read


def fit(filename):
    configs = read(filename + '@:')
    volumes = [a.get_volume() for a in configs]
    energies = [a.get_potential_energy() for a in configs]
    eos = EquationOfState(volumes, energies)
    v0, e0, B = eos.fit()
    return (4 * v0)**(1 / 3.0)


cutoffs = range(200, 801, 50)
a = [fit(f'Cu-{ecut}.txt') for ecut in cutoffs]
plt.figure(figsize=(6, 4))
plt.plot(cutoffs, a, 'o-')
plt.axis(ymin=3.60, ymax=3.62)
plt.xlabel('Plane-wave cutoff energy [eV]')
plt.ylabel('lattice constant [Ang]')
plt.savefig('Cu_conv_ecut.png')
```

↓エラーメッセージ
```
Traceback (most recent call last):
  File "analysis_ecut.py", line 18, in <module>
    a = [fit(f'Cu-{ecut}.txt') for ecut in cutoffs]
  File "analysis_ecut.py", line 18, in <listcomp>
    a = [fit(f'Cu-{ecut}.txt') for ecut in cutoffs]
  File "analysis_ecut.py", line 13, in fit
    v0, e0, B = eos.fit()
  File "/home/k0227/k022707/ASE/ase/lib64/python3.8/site-packages/ase/eos.py", line 201, in fit
    return self.fit_sjeos()
  File "/home/k0227/k022707/ASE/ase/lib64/python3.8/site-packages/ase/eos.py", line 319, in fit_sjeos
    raise ValueError('No minimum!')
ValueError: No minimum!
```

<br>

調べたところ`v0=None`となっているようなのですが、原因がわからないです。
