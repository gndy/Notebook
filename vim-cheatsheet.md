Vim 添加注释 详解
一、手注释动 vim下
1.插入注释：按Ctrl+v进入virtual模式
用上下键选中需要注释的行数
按大些“I”进入插入模式，输入注释符“#”，
然后立刻按下ESC**两下**
2.删除注释先按v进入列模式横向选中列的个数(如"/ /"注释符号,需要选中两列),
再按Esc 再按ctrl+v 进入列编辑模式,向下或向上移动光标按x、d键删除
二、快捷键注释
# sudo vim /etc/vim/vimrc
输入如下内容，保存
"F5 for comment
vmap <F5> :s=^\(//\)*=//=g<cr>:noh<cr>
nmap <F5> :s=^\(//\)*=//=g<cr>:noh<cr>
imap <F5> <ESC>:s=^\(//\)*=//=g<cr>:noh<cr>
"F6 for uncomment
vmap <F6> :s=^\(//\)*==g<cr>:noh<cr>
nmap <F6> :s=^\(//\)*==g<cr>:noh<cr>
imap <F6> <ESC>:s=^\(//\)*==g<cr>:noh<cr>
Vim 注释多行详细说明：
ctrl+v 进入列模式,向下或向上移动光标,把需要注释的行的开头标记起来,然后按大写的I,再插入注释符,比如#,再按Esc,就会全部注释。或者也可以运行下面这些命令：
:s/^/# #用"#"注释当前行
:2,50s/^ /# #在2~50行首添加"#"注释
:.,+3s/^/# #用"#"注释当前行和当前行后面的三行
:%s/^/# #用"#"注释所有行
顺便说一下vim的替换，这个常用，已经牢记，其实和上面用命令注释多行是一样的，只不过是上面注释的命令里的"^"符号代表开始位置而已，在下面这些命令中，"s"代表替换，part1代表查找的内容，part2代表替换的内容，"%"代表所有行，"g"代表替换整行里所有的内容（如果不加"/g"则只替换每行第一个匹配part1的地方）。
:s/part1/part2 #用part2替换当前行中第1个part1
:s/part1 /part2/g #用part2替换当前行中所有的part1
:%s/part1/part2 #用part2替换所有行中每行第1个part1
:%s/part1/part2/g #用part2替换所有行中所有的part1
:2,50s/part1 /part2 #用part2替换第2行到第50行中每行第1个part1
:2,50s/part1/part2/g #用 part2替换第2行到第50行中所有的part1
:.,+3s/part1/part2 #用part2替换当前行以及当前行后面的三行中每行第1个part1
:.,+3s/part1/part2/g #用part2替换当前行以及当前行后面的三行中所有的part1