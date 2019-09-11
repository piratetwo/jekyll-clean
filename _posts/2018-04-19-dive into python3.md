###chap5 正则表达式
正则表达式：
+ ^ 匹配字符串开始. $ 匹配字符串结尾
+ 检查罗马数字
 + 检查千位 pattern = '^M?M?M?$'
 + 检查百位 pattern = '^M?M?M?(CM|CD|D?C?C?C?)$'
 + pattern = '^M?M?M?(CM|CD|D?C?C?C?)(XC|XL|L?X?X?X?)$'
+ 使用语法{n,m}
 + pattern = '^M{0,3}$'
 + pattern = '^M{0,3}(CM|CD|D?C{0,3})(XC|XL|L?X{0,3})(IX|IV|V?I{0,3})$'

松散正则表达式
+ 空白符被忽略
+ 注释信息被忽略
+ ^ 匹配字符串开始位置
+ $ 匹配字符串结束位置
+ \b 匹配一个单词边界
+ \d 匹配一个数字
+ \D 匹配一个任意的非数字字符
+ x? 匹配可选的x字符。换句话说，就是0个或者1个x字符
+ x* 匹配0个或更多的x
+ x+ 匹配1个或者更多x
+ x{n,m} 匹配n到m个x，至少n个，不能超过m个
+ (a|b|c) 匹配单独的任意一个a或者b或者c
+ (x) 这是一个组，它会记忆它匹配到的字符串。你可以用re.search返回的匹配对象的groups()函数来获取到匹配的值
 
 ```python
>>> pattern = '''
    ^                   # beginning of string
    M{0,3}              # thousands - 0 to 3 Ms
    (CM|CD|D?C{0,3})    # hundreds - 900 (CM), 400 (CD), 0-300 (0 to 3 Cs),
                        #            or 500-800 (D, followed by 0 to 3 Cs)
    (XC|XL|L?X{0,3})    # tens - 90 (XC), 40 (XL), 0-30 (0 to 3 Xs),
                        #        or 50-80 (L, followed by 0 to 3 Xs)
    (IX|IV|V?I{0,3})    # ones - 9 (IX), 4 (IV), 0-3 (0 to 3 Is),
                        #        or 5-8 (V, followed by 0 to 3 Is)
    $                   # end of string
    '''
>>> re.search(pattern, 'M', re.VERBOSE)
<_sre.SRE_Match object at 0x008EEB48>
>>> re.search(pattern, 'MCMLXXXIX', re.VERBOSE)
<_sre.SRE_Match object at 0x008EEB48>
>>> re.search(pattern, 'MMMDCCCLXXXVIII', re.VERBOSE)
<_sre.SRE_Match object at 0x008EEB48>
>>> re.search(pattern, 'M')
```
+ 如果要使用松散正则表达式，需要传递一个叫re.VERBOSE的参数


匹配电话号码
```python
>>> phonePattern = re.compile(r'''
                # don't match beginning of string, number can start anywhere
    (\d{3})     # area code is 3 digits (e.g. '800')
    \D*         # optional separator is any number of non-digits
    (\d{3})     # trunk is 3 digits (e.g. '555')
    \D*         # optional separator
    (\d{4})     # rest of number is 4 digits (e.g. '1212')
    \D*         # optional separator
    (\d*)       # extension is optional and can be any number of digits
    $           # end of string
    ''', re.VERBOSE)
>>> phonePattern.search('work 1-(800) 555.1212 #1234').groups()
('800', '555', '1212', '1234')
>>> phonePattern.search('800-555-1212')
('800', '555', '1212', '')
```
###chap6 闭合与生成器
> 我们就是博格人。你们的语言和词源特性将会被添加到我们自己的当中。抵抗是徒劳的。

```python
import re

def build_match_and_apply_functions(pattern, search, replace):
    def matches_rule(word):                                     ①
        return re.search(pattern, word)
    def apply_rule(word):                                       ②
        return re.sub(search, replace, word)
    return (matches_rule, apply_rule)                           ③
```
+ ②在动态函数中使用外部参数值的技术称为 闭合【closures】

生成器
```python
def rules(rules_filename):
    with open(rules_filename, encoding='utf-8') as pattern_file:
        for line in pattern_file:
            pattern, search, replace = line.split(None, 3)
            yield build_match_and_apply_functions(pattern, search, replace)

def plural(noun, rules_filename='plural5-rules.txt'):
    for matches_rule, apply_rule in rules(rules_filename):
        if matches_rule(noun):
            return apply_rule(noun)
    raise ValueError('no matching rule for {0}'.format(noun))
```
###chap7 类迭代器
```python
class Fib:
    '''生成菲波拉稀数列的迭代器'''

    def __init__(self, max):
        self.max = max

    def __iter__(self):
        self.a = 0
        self.b = 1
        return self

    def __next__(self):
        fib = self.a
        if fib > self.max:
            raise StopIteration
        self.a, self.b = self.b, self.a + self.b
        return fib
```
+ 类 Class
+ 