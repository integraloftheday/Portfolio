---
title: Sample Blog Post
metaDescription: Test
date: 2019-01-01T00:00:00.000Z
author: Mason Manetta
summary: Test
tags:
  - Test
---
Congrats on finding this if you did. This is where the blog goes if I ever make one 

$$ f(x) = x^2 $$

``` js
function myFunction() {
  return true;
}
```

{% Notebook %}
# %% [markdown]
# dt-sine
Try editing this cell by clicking the pencil on the left!
# %%--- [python]
# properties:
#   run_on_load: true
#   top_hidden: true
# ---%%
import numpy as np
import matplotlib.pyplot as plt
import math

a = np.arange(-20,20,1)
b = np.sin(np.pi*a)

plt.clf()
plt.stem(b)
plt.show()
print(np.sin(math.pi))
{% endNotebook %}
