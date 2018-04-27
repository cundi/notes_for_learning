# unpack

`https://www.python.org/dev/peps/pep-0448/`

## update dict

- above python3.5

```python
combination = {**first_dictionary, "x": 1, "y": 2}
```

instead of:

```python
combination = first_dictionary.copy()
combination.update({"x": 1, "y": 2})
```

## append list

```bash
>>> ranges = [range(i) for i in range(5)]
>>> [*item for item in ranges]
[0, 0, 1, 0, 1, 2, 0, 1, 2, 3]

>>> {*item for item in ranges}
{0, 1, 2, 3}
```
