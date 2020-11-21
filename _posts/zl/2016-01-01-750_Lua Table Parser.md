---
layout: post
title: Lua Table Parser 
tags: [lua文章]
categories: [topic]
---
PyLuaTblParser类中有两个私有变量：

  1. `self.str_praser`是类`LuaStrParser`的实例，用来实现lua字符串的解析。
  2. `self.lua_table`是一个`dict`或者`list`，用来储存从lua table字符串或者其他dict中读取的数据。

## 接口

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    

|

    
    
    def (self, s):  
    读取Lua table数据，输入s为一个符合Lua table定义的字符串，无返回值  
    遇到Lua table格式错误则抛出异常  
      
    def dump(self):  
    根据类中数据返回Lua table字符串  
      
    def loadLuaTable(self, f):  
    从文件中读取Lua table字符串，f为文件路径，遇到Lua Table的格式错误则抛出异常，文件操作失败抛出异常  
      
    def dumpLuaTable(self, f):  
    将类中的内容以Lua table格式存入文件，f为文件路径，文件若存在则覆盖，文件操作失败抛出异常  
      
    def loadDict(self, d):  
    读取dict中的数据，存入类中，只处理数字和字符串两种类型的key，其他类型的key直接忽略  
      
    def dumpDict(self):  
    返回一个dict，包含类中的数据  
      
    def update(self, d):  
    用字典d更新类中的数据，类似于字典的update  
      
    PyLuaTblParser类支持用[]进行赋值、读写数据的操作，类似字典  
      
  
---|---  
  
# 代码

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33  
    34  
    35  
    36  
    37  
    38  
    39  
    40  
    41  
    42  
    43  
    44  
    45  
    46  
    47  
    48  
    49  
    50  
    51  
    52  
    53  
    54  
    55  
    56  
    57  
    58  
    59  
    60  
    61  
    62  
    63  
    64  
    65  
    66  
    67  
    68  
    69  
    70  
    71  
    72  
    73  
    74  
    75  
    76  
    77  
    78  
    79  
    80  
    81  
    82  
    83  
    84  
    85  
    86  
    87  
    88  
    89  
    90  
    91  
    92  
    93  
    94  
    95  
    96  
    97  
    98  
    99  
    100  
    101  
    102  
    103  
    104  
    105  
    106  
    107  
    108  
    109  
    110  
    111  
    112  
    113  
    114  
    115  
    116  
    117  
    118  
    119  
    120  
    121  
    122  
    123  
    124  
    125  
    126  
    127  
    128  
    129  
    130  
    131  
    132  
    133  
    134  
    135  
    136  
    137  
    138  
    139  
    140  
    141  
    142  
    143  
    144  
    145  
    146  
    147  
    148  
    149  
    150  
    151  
    152  
    153  
    154  
    155  
    156  
    157  
    158  
    159  
    160  
    161  
    162  
    163  
    164  
    165  
    166  
    167  
    168  
    169  
    170  
    171  
    172  
    173  
    174  
    175  
    176  
    177  
    178  
    179  
    180  
    181  
    182  
    183  
    184  
    185  
    186  
    187  
    188  
    189  
    190  
    191  
    192  
    193  
    194  
    195  
    196  
    197  
    198  
    199  
    200  
    201  
    202  
    203  
    204  
    205  
    206  
    207  
    208  
    209  
    210  
    211  
    212  
    213  
    214  
    215  
    216  
    217  
    218  
    219  
    220  
    221  
    222  
    223  
    224  
    225  
    226  
    227  
    228  
    229  
    230  
    231  
    232  
    233  
    234  
    235  
    236  
    237  
    238  
    239  
    240  
    241  
    242  
    243  
    244  
    245  
    246  
    247  
    248  
    249  
    250  
    251  
    252  
    253  
    254  
    255  
    256  
    257  
    258  
    259  
    260  
    261  
    262  
    263  
    264  
    265  
    266  
    267  
    268  
    269  
    270  
    271  
    272  
    273  
    274  
    275  
    276  
    277  
    278  
    279  
    280  
    281  
    282  
    283  
    284  
    285  
    286  
    287  
    288  
    289  
    290  
    291  
    292  
    293  
    294  
    295  
    296  
    297  
    298  
    299  
    300  
    301  
    302  
    303  
    304  
    305  
    306  
    307  
    308  
    309  
    310  
    311  
    312  
    313  
    314  
    315  
    316  
    317  
    318  
    319  
    320  
    321  
    322  
    323  
    324  
    325  
    326  
    327  
    328  
    329  
    330  
    331  
    332  
    333  
    334  
    335  
    336  
    337  
    338  
    339  
    340  
    341  
    342  
    343  
    344  
    345  
    346  
    347  
    348  
    349  
    350  
    351  
    352  
    353  
    354  
    355  
    356  
    357  
    358  
    359  
    360  
    361  
    362  
    363  
    364  
    365  
    366  
    367  
    368  
    369  
    370  
    371  
    372  
    373  
    374  
    375  
    376  
    377  
    378  
    379  
    380  
    381  
    382  
    383  
    384  
    385  
    386  
    387  
    388  
    389  
    390  
    391  
    392  
    393  
    394  
    395  
    396  
    397  
    398  
    399  
    400  
    401  
    402  
    403  
    404  
    405  
    406  
    407  
    408  
    409  
    410  
    411  
    412  
    413  
    414  
    415  
    416  
    417  
    418  
    419  
    420  
    421  
    422  
    423  
    424  
    425  
    426  
    427  
    428  
    429  
    430  
    431  
    432  
    433  
    434  
    435  
    436  
    437  
    438  
    439  
    440  
    

|

    
    
    class LuaStrParser:  
        __spaceChars = [' ', 't', 'n', 'r']  
        __stopChars = [' ', 't', 'n', 'r', '=', ',', ';', '}', ']']  
        keyTypes = [int, float, str]  
      
        __specialChars = {  
            'a': 'a', 'b': 'b', 'f': 'f', 'n': 'n', 'r': 'r', 't': 't', 'v': 'v',  
            '\': '\', '"': '"', "'": "'",  
        }  
      
        __reverseSpecialChars = {  
            'a': '\a', 'b': '\b', 'f': '\f', 'n': '\n', 'r': '\r', 't': '\t', 'v': '\v',  
            '\': '\\', '"': '\"', "'": "\'"  
        }  
      
        def __init__(self):  
            self.__lua_object = None  
            self.__lua_str = None  
      
        def __jumpSpace(self, string, loc):  
            while loc < len(string) and string[loc] in self.__spaceChars:  
                loc += 1  
            return loc  
      
        def __getBackSlashNum(self, lua_str, loc):  
            backslash_num = 0  
            while loc >= 0 and lua_str[loc] == '\':  
                loc -= 1  
                backslash_num += 1  
            return backslash_num  
      
        def __getStringEnd(self, lua_str, loc, quote):  
            result = loc + 1  
            if lua_str[loc] == quote:  
                end = lua_str.find(quote, loc + 1)  
                if end != loc + 1:  
                    at_end = False  
                    while not at_end:  
                        if self.__getBackSlashNum(lua_str, end - 1) % 2 == 0:  
                            at_end = True  
                        if not at_end:  
                            end = lua_str.find(quote, end + 1)  
                if end < 0:  
                    raise Exception("wrong quotes!")  
                result = end  
            return result  
      
        def __getStringConent(self, lua_str, loc):  
            """  
            return the string in the  @para lua_str  
            thee result contain the quotes"""  
            loc = self.__jumpSpace(lua_str, loc)  
            result = None  
            if lua_str[loc] == '"' or lua_str[loc] == "'":  
                end = self.__getStringEnd(lua_str, loc, lua_str[loc])  
                result = lua_str[loc: end + 1]  
                loc = end + 1  
            return result, loc  
      
        def __removeComment(self, lua_str):  
            result = ""  
            loc = 0  
            while loc < len(lua_str):  
                  
                if lua_str[loc] == '"' or lua_str[loc] == "'":  
                    string, loc = self.__getStringConent(lua_str, loc)  
                    result += string  
                # remove comment  
                elif lua_str[loc] == '-':  
                    if loc + 1 < len(lua_str) and lua_str[loc + 1] == '-':  
                        loc += 2  
                        # multi line comments  
                        if loc + 1 < len(lua_str) and lua_str[loc:loc + 2] == '[[':  
                            loc = lua_str.find(']]', loc + 1) + 2  
                        # single line comment  
                        else:  
                            loc = lua_str.find('n', loc + 1) + 1  
                        # add a space after remove the comment  
                        result += ' '  
                    else:  
                        result += lua_str[loc]  
                        loc += 1  
                # remove 'r'  
                elif lua_str[loc] == 'r':  
                    loc += 1  
                    continue  
                else:  
                    result += lua_str[loc]  
                    loc += 1  
            return result  
      
        def __getRightBrace(self, lua_str, loc):  
            loc = lua_str.find('{', loc) + 1  
            left_barce_num = 0  
            while loc < len(lua_str):  
                if lua_str[loc] == '"' or lua_str[loc] == "'":  
                    loc = self.__getStringEnd(lua_str, loc, lua_str[loc]) + 1  
                elif lua_str[loc] == '{':  
                    left_barce_num -= 1  
                    loc += 1  
                elif lua_str[loc] == '}':  
                    left_barce_num += 1  
                    if left_barce_num >= 1:  
                        break  
                    loc += 1  
                else:  
                    loc += 1  
            if loc == len(lua_str):  
                raise Exception("wrong number of brace")  
            return loc  
      
        def __getRightBracket(self, lua_str, loc):  
            loc = lua_str.find('[', loc) + 1  
            while loc < len(lua_str):  
                if lua_str[loc] == '"' or lua_str[loc] == "'":  
                    loc = self.__getStringEnd(lua_str, loc, lua_str[loc]) + 1  
                elif lua_str[loc] == ']':  
                    break  
                else:  
                    loc += 1<br