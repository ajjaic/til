# Using PDB debugger in a python script

To use the python debugger(PDB) that comes with python, open a script and do
```python
import pdb
```

and at that point where you want the debugger to trigger, insert
```python
pdb.set_trace()
```

The debugger can also be invoked in a single line, as follows.
```python
import pdb; pdb.set_trace()
```

# Check for syntax errors

To check for compile errors,
```bash
python -m py_compile script.py
```

[source](http://stackoverflow.com/questions/4284313/how-can-i-check-the-syntax-of-python-script-without-executing-it)

# Multiline Strings and Multiline statements/expressions
## Multiline Strings
```Python
s = ("this is a very"
     "long string too"
     "for sure ..."
    )
print(s)
```

And this is what you get,
```
this is a verylong string toofor sure
```

It doesn't add any `spaces` or anything. You gotta add it yourself.
[source](http://stackoverflow.com/questions/10660435/pythonic-way-to-create-a-long-multi-line-string)

## Multiline Statements/Expressions
### Implicit
The preferred way is to do it the implicit way. Expressions inside parentheses
can be entered in multiple lines. Python implicitly knows that everything
within the parentheses is part of the same statement/expression
```Python
a = some_func(
    '1' + '2' + '3' + '4')
```

### Explicit
The explicit way is to use the `\` symbol at the point where the line is
continued on the next line
```Python
a = '1' + '2' + \
    '3' + '4'
```

[source](http://stackoverflow.com/questions/4172448/is-it-possible-to-break-a-long-line-to-multiple-lines-in-python)
