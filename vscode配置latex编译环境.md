# vscode中配置latex编写环境
系统环境：Windows10


从[博客](https://zhuanlan.zhihu.com/p/38178015)中可知详细步骤

综合该文章的步骤，所有需要的配置如下，写入vscode配置文件就行（注意vscode的安装目录和pdf阅读器应用程序的安装位置需要换成自己的）：
```json
"latex-workshop.latex.tools": [
        {
            // 编译工具和命令
            "name": "xelatex",
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "-pdf",
                "%DOCFILE%"
            ]
        },
        {
            "name": "pdflatex",
            "command": "pdflatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOCFILE%"
            ]
        },
         {
            "name": "latex",
            "command": "latex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOCFILE%"
            ]
        },
        {
            "name": "dvips",
            "command": "dvips",
            "args": [
                "%DOCFILE%"
            ],
            "env": {}
        },
        {
            "name": "ps2pdf",
            "command": "ps2pdf",
            "args": [
                "%DOCFILE%"
            ],
            "env": {}
        },
        {
            "name": "bibtex",
            "command": "bibtex",
            "args": [
                "%DOCFILE%"
            ]
        }
    ],

    "latex-workshop.latex.recipes": [
        {
            "name": "pdf->bib->pdf->pdf",
            "tools": [
                "pdflatex",
                "bibtex",
                "pdflatex",
                "pdflatex"
            ]
     
        },
          {
            "name": "la->bib->la->la->dvi->ps->pdf",
            "tools": [
                "latex",
                "bibtex",
                "latex",
                "latex",
                "dvips",
                "ps2pdf"
            ]
        },
        {
            "name": "pdflatex",
            "tools": [
                "pdflatex"
            ]
        },
        {
            "name": "xe->bib->xe->xe",
            "tools": [
                "xelatex",
                "bibtex",
                "xelatex",
                "xelatex"
            ]
        },
        {
            "name": "xelatex",
            "tools": [
                "xelatex"
            ],
        }
    ],

"latex-workshop.view.pdf.viewer": "external",

"latex-workshop.view.pdf.external.viewer.command": "D:/software/SumatraPDF/SumatraPDF.exe",
"latex-workshop.view.pdf.external.viewer.args": [
    "-forward-search",
    "%TEX%",
    "%LINE%",
    "-reuse-instance",
    "-inverse-search",
    "\"D:/software/vscode/Microsoft VS Code/Code.exe\" \"D:/software/vscode/Microsoft VS Code/resources/app/out/cli.js\" -gr \"%f\":\"%l\"",
    "%PDF%"
],

"latex-workshop.view.pdf.external.synctex.command": "D:/software/SumatraPDF/SumatraPDF.exe",
"latex-workshop.view.pdf.external.synctex.args": [
    "-forward-search",
    "%TEX%",
    "%LINE%",
    "-reuse-instance",
    "-inverse-search",
    //下面这行是正向搜索，从tex至pdf
    "\"D:/software/vscode/Microsoft VS Code/Code.exe\" \"D:/software/vscode/Microsoft VS Code/resources/app/out/cli.js\" -gr \"%f\":\"%l\"",
    //下面这行是反向搜索，从pdf至tex
    "\"D:/software/vscode/Microsoft VS Code/Code.exe\" \"D:/software/vscode/Microsoft VS Code/resources/app/out/cli.js\" -g \"%f\":\"%l\"",
    "%PDF%",
],
"latex-workshop.latex.clean.fileTypes": [
     //清楚以下格式文件
        "*.aux",
        "*.bbl",
        "*.blg",
        "*.dvi",
        "*.idx",
        "*.ind",
        "*.lof",
        "*.lot",
        "*.out",
        "*.toc",
        "*.acn",
        "*.acr",
        "*.alg",
        "*.glg",
        "*.glo",
        "*.gls",
        "*.ist",
        "*.fls",
        "*.log",
        "*.fdb_latexmk",
        ],
```

同时，以下代码用于设置键盘快捷键，打开keyjson键盘设置：
```json
{
    "key": "alt+s",
    "command": "latex-workshop.synctex",
    "when": "editorTextFocus && !isMac"
},
{
    "key": "alt+b",
    "command": "latex-workshop.build",
    "when": "editorTextFocus && !isMac"
},
{
    "key": "alt+t",
    "command": "latex-workshop.kill",
    "when": "editorTextFocus && !isMac"
},
{
    "key": "alt+e",
    "command": "latex-workshop.recipes"
},
//这段代码的意义是将 Alt+s 绑定到正向搜索，将 Alt+b 绑定到使用默认 recipe 编译，将 Alt+t 绑定到终止编译，将 Alt+e 绑定到选择其他 recipe 编译，可以自行更换为适合自己的快捷键，只需修改“key”那一项即可
 
```
