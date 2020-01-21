# NVMe Write Test

There are NVMe cards on TigerGPU and Traverse. Not Della, Perseus or Adroit. They are ony useful when the block size
of the write is sufficiently high in comparison to the other filesystems.

```python
import numpy as np
from time import perf_counter

cluster='Tiger'
#cluster='Della (cascade)'
paths = ['/tmp', '/scratch', '/home/jdh4', '/scratch/gpfs/jdh4', '/tigress/jdh4']
myfile = '4GB.dat'

# create a NumPy array of 500K double precision numbers (4 GB)
N = 20_000_000
x = np.random.random(N)

# write the array to the different filesystems are record write times
results = []
for path in paths:
  t0 = perf_counter()
  np.savetxt(path + '/' + myfile, x, fmt='%1.6f')
  #np.save(path + '/' + myfile, x)
  results.append((path, perf_counter() - t0))
print(results)

# remove written files
import os
for path in paths:
  fullpath = path + '/' + myfile
  #fullpath = path + '/' + myfile + '.npy'
  if os.path.exists(fullpath): os.remove(fullpath)

# create plot of the results
import matplotlib.pyplot as plt
plt.style.use('~/grs/halverson.mplstyle')
mypaths, times = zip(*results)
assert paths == list(mypaths)

ints = range(len(times))
plt.barh(ints, times)
plt.yticks(ints, mypaths, fontsize=10)
plt.xlabel('time (s)')
plt.title('Time required to write a 4 GB NumPy array to ASCII file using np.savetxt() on ' + cluster)
plt.tight_layout()
plt.savefig('write_times.png')
```

To run this code:

```
$ module load anaconda3
$ python write_test.py
```
