# PL0
编译原理期末项目

## Part 1
### 实验要求
* 修改资料中所给的PL0编译程序，使之能够编译给定的PL0源程序，并将中间代码和栈顶结果输出到文件中。

### 实验流程
#### 环境配置
* 安装Free Pascal
#### 编译程序
* 首先，我将资料中给定的编译程序复制下来，直接在命令行使用*fpc*命令进行编译，结果出错。经过仔细对比，因为Word排版的问题，有很多符号方面的错误，例如“非/取反”符号直接复制到Pascal代码中会消失，这也是造成了我后续很多错误的一个主要原因。其次，还有“小于等于/大于等于”，复制下来的代码都是有错误的，不是编译器能够编译的符号，所以我得将其拆分为"<"和"="这种形式。
* 接着，编译程序还是会出问题，因为编译程序中虽然有输出语句，但是并没有读入程序的地方，所以需要在最后进行读入。其次，因为需要将结果输出到文件中，所以我在一开始打开了一个输出文件*result.txt*，然后在代码中的每一句write后都加一句，将结果输出到文件中。当然，需要在一开始声明变量。
```pascal
//code
  input : text;
  output : text;
  sfile: string; 
  //code
Begin  

  writeln('Please input source program file name : ');
  
  readln(sfile);    
  assign(input,sfile);   
  assign(output, 'result.txt'); 
  reset(input);    
  rewrite(output);
  //code

  writeln;    
  close(input);    
  close(output);
  readln(sfile);    
End.1
``` 
* 修改之后，依然是不能够运行，经过调试后，问题出现在*condition*部分，其处理的情况太少了，于是我根据代码所需，补充了剩下的一部分，完成了编译程序。编译，能够正常运行。

### 运行结果
* 详见*result.txt*。


## Part 2
### 实验要求
* 在上次的实验的基础上，为编译程序添加*Write*与*Read*语句，使得PL0源程序能够使用*Write*和*Read*语句，将结果输出到文件中。
### 实验流程
#### 环境配置
* 同 Part 1。
#### 编译程序
* 首先，将编译程序头的一个变量从11变为13，因为要添加两条命令。
```pascal
norw = 13;
```
* 同时，也要修改symbol和fct，添加*write*与*read*命令。
```pascal
Type
  symbol = (nul, ident, number, plus, minus, times, slash, oddsym,
            eql, neq, lss, leq, gtr, geq, lparen, rparen, comma, semicolon,
            period, becomes, beginsym, endsym, ifsym, thensym,
            whilesym, dosym, callsym, constsym, varsym, procsym,readsym,writesym );
  alfa = packed array [1..al] Of char;
  objectType = (constant, variable, procedures);
  symset = set Of symbol;
  fct = (lit, opr, lod, sto, cal, int, jmp, jpc, red, wrt); 
```
* 再者，在*condition*中添加*wirte*与*read*语句，这是最为重要的一部分。
```pascal
  Else If sym = readsym    
         Then
         Begin
           getsym;    
           If sym = lparen    
             Then
             Repeat    
               getsym;    
               If sym = ident    
                 Then
                 Begin
                   i := position(id);
                   
                   If i = 0
                   
                     Then error(11)    
                   Else If table[i].kind <> variable
                           
                          Then
                          Begin
                            error(12);
                            
                            i := 0    
                          End
                   Else With table[i] Do    
                          gen(red,lev-level,adr)
                          
                 End
               Else error(4);
               
               getsym;    
             Until sym <> comma
                   
           Else error(40);
           
           If sym <> rparen    
             Then error(22);    
           getsym    
         End
  Else If sym = writesym    
         Then
         Begin
           getsym;    
           If sym = lparen    
             Then
             Begin
               Repeat    
                 getsym;    
                 expression([rparen,comma]+fsys);
                 
                 gen(wrt,0,0);    
               Until sym <> comma;    
               If sym <> rparen    
                 Then error(22);    
               getsym    
             End
           Else error(40)    
         End;
```
* 最后，在主程序中修改*word*，*wsym*以及*mnemonic*，添加两个条目，具体如下：
```pascal
Begin  

  writeln('Please input source program file name : ');
  
  readln(sfile);    
  assign(input,sfile);   
  assign(output, 'result.txt'); 
  reset(input); 
  rewrite(output);   
  For ch := 'A' To ';' Do
    ssym[ch] := nul;
  word[1] := 'begin        ';
  word[2] := 'call         ';
  word[3] := 'const        ';
  word[4] := 'do           ';
  word[5] := 'end          ';
  word[6] := 'if           ';
  word[7] := 'odd          ';
  word[8] := 'procedure    ';
  word[9] := 'read         ';
  word[10] := 'then         ';
  word[11] := 'var          ';
  word[12] := 'while        ';
  word[13] := 'write        ';
  

  wsym[1] := beginsym;
  wsym[2] := callsym;
  wsym[3] := constsym;
  wsym[4] := dosym;
  wsym[5] := endsym;
  wsym[6] := ifsym;
  wsym[7] := oddsym;
  wsym[8] := procsym;
  wsym[9] := readsym;
  wsym[10] := thensym;
  wsym[11] := varsym;
  wsym[12] := whilesym;
  wsym[13] := writesym; 

  ssym['+'] := plus;
  ssym['-'] := minus;
  ssym['*'] := times;
  ssym['/'] := slash;
  ssym['('] := lparen;
  ssym[')'] := rparen;
  ssym['='] := eql;
  ssym[','] := comma;
  ssym['.'] := period;
  ssym['<'] := lss;
  ssym['>'] := gtr;
  ssym[';'] := semicolon;

  mnemonic[lit] := 'LIT';
  mnemonic[opr] := 'OPR';
  mnemonic[lod] := 'LOD';
  mnemonic[sto] := 'STO';
  mnemonic[cal] := 'CAL';
  mnemonic[int] := 'INT';
  mnemonic[jmp] := 'JMP';
  mnemonic[jpc] := 'JPC';
  mnemonic[red] := 'RED  ';
  mnemonic[wrt] := 'WRT  '; 
  declbegsys := [constsym, varsym, procsym];
  statbegsys := [beginsym, callsym, ifsym, whilesym];
  facbegsys := [ident, number, lparen];

  cc := 0;
  cx := 0;
  ll := 0;
  ch := ' ';
  kk := al;
  getsym;
  block(0, 0, [period]+declbegsys+statbegsys);
  If sym <> period Then error(9);
  If err = 0 Then interpret
  Else write('ERRORS IN PL/0 PROGRAM');
  writeln;    
  close(input);    
  close(output);
  readln(sfile);    
End.1
```

### 实验结果
* 详见*result.txt*。