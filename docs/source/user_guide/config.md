- [Config](#config)
  - [Basic](#basic)
  - [Substitution](#substitution)
    - [Callable Substitution](#callable-substitution)
    - [Substitution by Path](#substitution-by-path)
    - [Substitution by Attribute](#substitution-by-attribute)
    - [Default Value by Path Substitution](#default-value-by-path-substitution)
    - [Ignore Substitution](#ignore-substitution)
    - [Lazy Evaluation](#lazy-evaluation)


# Config

## Basic
```python
from pytorch_pfn_extras.config import Config
import yaml
pre_eval_config = yaml.load('''
foo:
  bar: 'bar_value'
  ls:
    - 'first'
    - key0: 'value0'
      key1: 'value1'
baz: 'baz_value'
''')
config = Config(pre_eval_config)
```

Accessing config values:
```python
print(config['/foo/ls/0'])
# 'first'
print(config['/foo/ls/1/key0'])
# 'value0'
print(config['/foo/ls'])
# ['first', {'key0': 'value0', 'key1': 'value1'}]
print(config['/baz'])
# 'baz_value'
```

## Substitution

### Callable Substitution

You could replace a value as the return value of a callable.
- `types` is an additional input to `Config`. `types` is a mapping from a callable's name to the actual callable.
- A sub-dictionary containing the key `type` invokes callable substitution.

```python
pre_eval_config = yaml.load('''
name:
  type: concat
  x0: 'First'
  x1: 'Last'
''')

types = {
  'concat': lambda x0, x1: x0 + ' ' + x1
}

config = Config(pre_eval_config, types)
# the value returned by
# concat(x0='First', x1='Last')
print(config['/name'])
# 'First Last'
```

#### Nested

```python
pre_eval_config = yaml.load('''
name:
  type: concat
  x0: 'First'
  x1:
    type: concat
    x0: 'Middle'
    x1: 'Last'
''')
types = {
  'concat': lambda x0, x1: x0 + ' ' + x1
}
config = Config(pre_eval_config, types)
print(config['/name'])
# First Middle Last
```

#### Class

```python
pre_eval_config = yaml.load('''
dataset:
  type: Dataset
  n_class: 10
''')

class Dataset(object):

    def __init__(self, n_class):
        self.n_class = n_class

types = {
  'Dataset': Dataset,
}

config = Config(pre_eval_config, types)
print(isintance(config['/dataset'], Dataset))
# True
```

### Substitution by Path
#### Absolute
`@/absolute/path` is replaced by the value at `/absolute/path`.

```python
pre_eval_config = yaml.load('''
foo: 'FOO'
boo:
  baz: '@/foo'
''')
config = Config(pre_eval_config)
print(config['/boo/baz'])
# FOO
```

#### Relative
Relative path is also possible using `@relative/path`.

```python
pre_eval_config = yaml.load('''
foo: 'FOO'
boo:
  baz: '@../foo'
''')
config = Config(pre_eval_config)
print(config['/boo/baz'])
# FOO
```

### Substitution by Attribute

`@/path/to/obj.attr_name` is replaced by:
1. Use substitution by path to get an object at `/path/to/obj`.
2. Replace the config value by `getattr(obj, attr_name)`, where `obj` is obtained at step 1.

```python
pre_eval_config = yaml.load('''
dataset:
  type: Dataset
  n_class: 10
n_data: '@/dataset.n_data'
''')

class Dataset(object):

    def __init__(self, n_class):
        self.n_class = n_class
        self.n_data = 4

types = {
  'Dataset': Dataset,
}

config = Config(pre_eval_config, types)
print(config['/n_data'])
# 4
```

### Default Value by Path Substitution
`customize_type` is a decorator that sets default argument values by path substitution.

```python
from pytorch_pfn_extras.config import customize_type

pre_eval_config = yaml.load('''
dataset:
  type: Dataset
n_class: 5
''')

# If n_class is not passed, the value would be config['/n_class'].
# Both absolute and relative paths are allowed.
@customize_type(n_class='/n_class')
class Dataset(object):

    def __init__(self, n_class):
        self.n_class = n_class

types = {
  'Dataset': Dataset,
}

config = Config(pre_eval_config, types)
print(config['/dataset'].n_class)
# 5
```

### Ignore Substitution

Access using `config['!/path']` instead of `config['/path']`.

```python
pre_eval_config = yaml.load('''
name:
  type: concat
  x0: 'First'
  x1: 'Last'
''')

types = {
  'concat': lambda x0, x1: x0 + ' ' + x1
}

config = Config(pre_eval_config, types)
print(config['!/name'])
# {'type': 'concat', 'x0': 'First', 'x1': 'Last'}
```

### Lazy Evaluation
Callable substitution is lazily executed.
This means that callables that are not dependent on the accesed value do not get executed.

```python
pre_eval_config = yaml.load('''
foo:
  - type: f0
  - '@/bar'
bar:
  type: f1
baz:
  type: f2
''')

def f0():
    print('f0 called')
    return 'f0_return'

def f1():
    print('f1 called')
    return 'f1_return'

def f2():
    print('f2 called')
    return 'f2_return'

types = {
  'f0': f0,
  'f1': f1,
  'f2': f2,
}

config = Config(pre_eval_config, types)
config['/foo']  # f2 does not get called
# f0 called
# f1 called
```

