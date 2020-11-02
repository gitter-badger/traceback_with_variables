## Python traceback (stacktrace) printing variables.

Very simple to use, but versatile when needed.    

---

![Example](https://raw.githubusercontent.com/andy-landy/traceback_with_variables/master/header.png)

---

### 
###
###

### Quick Start

Simplest usage:
```python
    from traceback_with_variables import activate_by_import
```

Decorator:
```python
    @prints_tb
    # def main(): or def some_func(...):
```

Context:
```python
    with printing_tb():
```

Work with traceback lines:
```python
    return '\n'.join(iter_tb_lines(e))
```

Using a logger:
```python
    with printing_tb(file_=LoggerAsFile(logger)):
    # or
    @prints_tb(file_=LoggerAsFile(logger)): 
```

### Installation

```
pip install traceback-with-variables
```

### Rationale

* Tired of putting all your variables in debug exception messages? Just stop it. Go clean your code:

    ```diff
      def main():
          sizes_str = sys.argv[1]
          h1, w1, h2, w2 = map(int, sizes_str.split())
    -     try:
              return get_avg_ratio([h1, w1], [h2, w2])
    -     except:
    -         logger.error(f'something happened :(, variables = {variables()[:1000]}')
    -         raise
    -         # or
    -         raise MyToolException(f'something happened :(, variables = {variables()[:1000]}')
              
      def get_avg_ratio(size1, size2):
    -     try:
              return mean(get_ratio(h, w) for h, w in [size1, size2])
    -     except:
    -         logger.error(f'something happened :(, size1 = {size1}, size2 = {size2}')
    -         raise
    -         # or
    -         raise MyToolException(f'something happened :(, size1 = {size1}, size2 = {size2}')
    
      def get_ratio(height, width):
    -     try:
              return height / width
    -     except:
    -         logger.error(f'something happened :(, width = {width}, height = {height}')
    -         raise
    -         # or
    -         raise MyToolException(f'something happened :(, width = {width}, height = {height}')
    ```
    Must become this:
    
    ```diff
    + from traceback_with_variables import activate_by_import
    
      def main():
          sizes_str = sys.argv[1]
          h1, w1, h2, w2 = map(int, sizes_str.split())
          return get_avg_ratio([h1, w1], [h2, w2])
              
      def get_avg_ratio(size1, size2):
          return mean(get_ratio(h, w) for h, w in [size1, size2])
    
      def get_ratio(height, width):
          return height / width
    ```
    
    To produce:
    
    ```
    Traceback with variables (most recent call last):
      File "./temp.py", line 7, in main
        return get_avg_ratio([h1, w1], [h2, w2])
          sizes_str = '300 200 300 0'
          h1 = 300
          w1 = 200
          h2 = 300
          w2 = 0
      File "./temp.py", line 10, in get_avg_ratio
        return mean([get_ratio(h, w) for h, w in [size1, size2]])
          size1 = [300, 200]
          size2 = [300, 0]
      File "./temp.py", line 10, in <listcomp>
        return mean([get_ratio(h, w) for h, w in [size1, size2]])
          .0 = <tuple_iterator object at 0x7ff61e35b820>
          h = 300
          w = 0
      File "./temp.py", line 13, in get_ratio
        return height / width
          height = 300
          width = 0
    builtins.ZeroDivisionError: division by zero
    ```

* Make automated logging easier:
    
    ```python
    logger = logging.getLogger('main')
    
    def main():
        ...
        with printing_tb(file_=LoggerAsFile(logger))
            ...
    ```
    
    ```
    2020-03-30 18:24:31 main ERROR Traceback with variables (most recent call last):
    2020-03-30 18:24:31 main ERROR   File "./temp.py", line 7, in main
    2020-03-30 18:24:31 main ERROR     return get_avg_ratio([h1, w1], [h2, w2])
    2020-03-30 18:24:31 main ERROR       sizes_str = '300 200 300 0'
    2020-03-30 18:24:31 main ERROR       h1 = 300
    2020-03-30 18:24:31 main ERROR       w1 = 200
    2020-03-30 18:24:31 main ERROR       h2 = 300
    2020-03-30 18:24:31 main ERROR       w2 = 0
    2020-03-30 18:24:31 main ERROR   File "./temp.py", line 10, in get_avg_ratio
    2020-03-30 18:24:31 main ERROR     return mean([get_ratio(h, w) for h, w in [size1, size2]])
    2020-03-30 18:24:31 main ERROR       size1 = [300, 200]
    2020-03-30 18:24:31 main ERROR       size2 = [300, 0]
    2020-03-30 18:24:31 main ERROR   File "./temp.py", line 10, in <listcomp>
    2020-03-30 18:24:31 main ERROR     return mean([get_ratio(h, w) for h, w in [size1, size2]])
    2020-03-30 18:24:31 main ERROR       .0 = <tuple_iterator object at 0x7ff412acb820>
    2020-03-30 18:24:31 main ERROR       h = 300
    2020-03-30 18:24:31 main ERROR       w = 0
    2020-03-30 18:24:31 main ERROR   File "./temp.py", line 13, in get_ratio
    2020-03-30 18:24:31 main ERROR     return height / width
    2020-03-30 18:24:31 main ERROR       height = 300
    2020-03-30 18:24:31 main ERROR       width = 0
    2020-03-30 18:24:31 main ERROR builtins.ZeroDivisionError: division by zero
    ```

* Free your exceptions of unnecessary information load:
    
    ```python
    def make_a_cake(sugar, eggs, milk, flour, salt, water):
        is_sweet = sugar > salt
        is_vegan = not (eggs or milk)
        is_huge = (sugar + eggs + milk + flour + salt + water > 10000)
        if not (is_sweet or is_vegan or is_huge):
            raise ValueError('This is unacceptable, guess why!')
        ...
    ```

* Stop this tedious practice in production:
   
     step 1: Notice some exception in a production service. \
     step 2: Add more printouts, logging, and exception messages. \
     step 3: Rerun the service. \
     step 4: Wait till (hopefully) the bug repeats. \
     step 5: Examine the printouts and possibly add some more info (then go back to step 2). \
     step 6: Erase all recently added printouts, logging and exception messages. \
     step 7: Go back to step 1 once bugs appear.

### Reference

---

#### All functions have output customization
* `max_value_str_len` max length of each variable string
* `max_exc_str_len` max length of exception, should variable print fail
* `num_context_lines` number of lines around the target code line to print
* `ellipsis_` string to denote long strings truncation, default=`...`
* `file_` where to print exception, a file or a wrapped logger, default=`sys.stderr` i.e. usual printing to console

---

#### `activate_by_import`
Just import it. No arguments, for real quick use.
```python
from traceback_with_variables import activate_by_import
```

---

#### `override_print_tb`
Call once in the beginning of your program, to change how traceback after an uncaught exception looks.
```python
def main():
    override_print_tb(...)
```

---

#### `prints_tb`
Function decorator, used for logging or simple printing of scoped tracebacks with variables. I.e. traceback is shorter as it includes only frames inside the function call. Program exiting due to unhandled exception still prints a usual traceback.
```python
@prints_tb
def f(...):

@prints_tb(...)
def f(...):
```

---

#### `printing_tb`
Context manager (i.e. `with ...`), used for logging or simple printing of scoped tracebacks with variables. I.e. traceback is shorter as it includes only frames inside the `with` scope. Program exiting due to unhandled exception still prints a usual traceback.
```python
with printing_tb(...):
```

---

#### `LoggerAsFile`
A logger-to-file wrapper, to pass a logger to print tools as a file.

---

#### `iter_tb_lines`
Iterates the lines, which are usually printed one-by-one in terminal.

### Recipes

#### Simplest usage
```python
from traceback_with_variables import activate_by_import
```

#### Override standard traceback if env variable X is set
```python
def main():
    override_print_tb(activate_by_env_var='X')
```

#### Override standard traceback unless env variable Y is set
```python
def main():
    override_print_tb(deactivate_by_env_var='Y')
```

#### Log traceback for a block of code, exclude external frames
```python
    logger = logging.getLogger(__name__)
    ...
    with printing_tb(file_=LoggerAsFile(logger)):
```

#### Log traceback for a function, exclude external frames
```python
    logger = logging.getLogger(__name__)
    ...
    @prints_tb(file_=LoggerAsFile(logger))
    def f(...):
```

#### Print traceback for inner frames
```python
    with printing_tb():
```
