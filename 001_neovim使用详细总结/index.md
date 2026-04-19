# Neovim 使用详细总结


# Neovim 详细使用总结
Neovim 是 Vim 的社区驱动现代化分支，**100%兼容Vim核心模态编辑逻辑与绝大多数Vimscript配置/插件**，同时从底层重构了异步非阻塞架构，原生支持Lua脚本、内置LSP客户端、Tree-sitter语法解析、终端模拟器等现代IDE核心能力，兼顾了模态编辑的极致效率与现代开发的全场景需求，是当前终端编辑器的主流选择之一。

## 一、核心特性与Vim的核心差异
| 特性维度 | Neovim | Vim |
| :--- | :--- | :--- |
| 核心架构 | 原生异步非阻塞，耗时任务（LSP、格式化、搜索）完全不阻塞主界面 | 同步阻塞为主，Vim8后仅支持有限异步能力，体验割裂 |
| 脚本支持 | Lua为一等公民，执行效率高、语法简洁、生态繁荣，同时兼容Vimscript | 原生仅支持Vimscript，Lua支持为第三方扩展，能力受限 |
| 开发模式 | 社区驱动，迭代快，持续拥抱现代开发标准 | 原作者主导，更新保守，优先保证向下兼容 |
| 内置能力 | 原生内置LSP客户端、Tree-sitter、终端模拟器、浮动窗口 | 无原生LSP/Tree-sitter，终端能力弱，UI扩展受限 |
| 配置规范 | 遵循XDG基础目录规范，配置文件管理更清晰规范 | 传统vimrc路径，无统一目录规范 |

### 核心杀手锏特性
1.  **异步IO架构**：彻底解决Vim时代插件/外部命令导致的编辑器卡顿问题，是流畅支持LSP、代码格式化等现代功能的核心基础。
2.  **原生Lua支持**：替代晦涩的Vimscript，配置更易读、易维护、执行更快，整个社区生态已全面转向Lua开发。
3.  **内置LSP客户端**：原生支持语言服务器协议，无需第三方插件即可实现代码补全、跳转定义、悬停文档、重构、诊断等IDE级能力。
4.  **Tree-sitter集成**：基于AST抽象语法树实现精准的语法高亮、增量选择、代码折叠、智能缩进，远优于传统正则匹配的语法解析。
5.  **内置终端模拟器**：无需离开编辑器即可打开终端，支持分屏、浮动窗口，完全兼容Neovim的按键映射与模式。

## 二、安装与快速入门
### 1. 全平台安装
#### Linux
```bash
# Ubuntu/Debian
sudo apt install neovim
# Fedora/RHEL
sudo dnf install neovim
# Arch Linux
sudo pacman -S neovim
```

#### macOS
```bash
# Homebrew
brew install neovim
```

#### Windows
```powershell
# Scoop
scoop install neovim
# Chocolatey
choco install neovim
# Winget
winget install Neovim.Neovim
```

安装完成后，执行 `nvim --version` 验证安装成功，执行 `nvim` 即可启动编辑器。

### 2. 极简入门操作
Neovim完全兼容Vim的模态编辑逻辑，核心6种模式，新手只需先掌握3种核心模式：
| 模式 | 进入方式 | 核心用途 | 退出方式 |
| :--- | :--- | :--- | :--- |
| 普通模式(Normal) | 启动默认进入，其他模式按`Esc`返回 | 光标移动、文本操作、命令执行 | 无需退出，基础模式 |
| 插入模式(Insert) | 普通模式按`i`/`a`/`o`等 | 文本输入、编辑 | 按`Esc`返回普通模式 |
| 命令模式(Command) | 普通模式按`:` | 执行保存、退出、搜索、设置等命令 | 执行完命令自动返回，或按`Esc`取消 |

#### 新手必记基础命令
```vim
" 命令模式核心
:w  保存文件
:q  退出编辑器
:wq 保存并退出
:q! 强制退出不保存
:help 打开内置帮助文档（学习Neovim最权威的资源）

" 普通模式高频操作
i  在光标前进入插入模式
a  在光标后进入插入模式
o  在当前行下方新建行并进入插入模式
dd 删除当前行
yy 复制当前行
p  粘贴复制/删除的内容
u  撤销上一步操作
Ctrl+r 重做撤销的操作
```

## 三、核心基础操作（模态编辑核心）
Neovim完全兼容Vim的操作逻辑，这里提炼高频核心操作，覆盖90%日常使用场景。

### 1. 光标移动（普通模式）
高效移动是模态编辑的核心，告别纯方向键移动，掌握以下高频操作：
```vim
" 基础移动
h 左  j 下  k 上  l 右

" 单词级移动
w  跳到下一个单词开头
e  跳到当前/下一个单词结尾
b  跳到当前/上一个单词开头

" 行级移动
0  跳到行首
^  跳到行首第一个非空字符
$  跳到行尾
gg 跳到文件开头
G  跳到文件结尾
:n 跳到第n行（如:10跳到第10行）

" 段落/屏幕级移动
Ctrl+f 向下翻一屏
Ctrl+b 向上翻一屏
Ctrl+d 向下翻半屏
Ctrl+u 向上翻半屏
zz 将当前行置于屏幕中央
```

### 2. 文本编辑（普通模式）
Neovim的编辑逻辑是「操作符+范围+对象」，掌握后可实现精准高效的文本处理。
#### 基础编辑操作
```vim
" 基础删除
x  删除光标所在字符
dd 删除当前行
d$ 删除光标到行尾的内容
d0 删除光标到行首的内容

" 基础复制粘贴
yy 复制当前行
yw 复制当前光标到单词结尾的内容
p  粘贴到光标后
P  粘贴到光标前

" 高级编辑（操作符+文本对象）
dw 删除一个单词
diw 删除光标所在的整个单词（不包含空格）
daw 删除光标所在的单词（包含空格）
di( 删除括号内的内容（不包含括号）
da( 删除括号及括号内的所有内容
" 同理：ci( 替换括号内内容，yi( 复制括号内内容，支持()、[]、{}、''、""、<>等
```

### 3. 搜索与替换
```vim
" 基础搜索
/关键词  向下搜索关键词
?关键词  向上搜索关键词
n  跳到下一个匹配结果
N  跳到上一个匹配结果
*  向下搜索光标所在的单词
#  向上搜索光标所在的单词

" 全局替换（命令模式）
:%s/旧内容/新内容/g  全局替换所有匹配项
:%s/旧内容/新内容/gc 全局替换，每次替换前确认
:10,20s/旧内容/新内容/g  替换10-20行内的匹配项
```

### 4. 窗口、缓冲区与标签页管理
#### 窗口分屏
```vim
:sp 文件名  水平分屏打开文件
:vs 文件名  垂直分屏打开文件
Ctrl+w + h/j/k/l  切换左/下/上/右的窗口
Ctrl+w + =  所有窗口等宽等高
Ctrl+w + q  关闭当前窗口
```

#### 缓冲区管理
缓冲区是Neovim打开的文件内存对象，分屏只是窗口展示，核心是缓冲区操作：
```vim
:ls  列出所有打开的缓冲区
:b 缓冲区编号/文件名  切换到指定缓冲区
:bd 关闭当前缓冲区
:bn 切换到下一个缓冲区
:bp 切换到上一个缓冲区
```

#### 标签页管理
每个标签页可包含多个窗口，用于区分不同工作场景：
```vim
:tabe 文件名  新建标签页打开文件
gt  切换到下一个标签页
gT  切换到上一个标签页
:tabclose 关闭当前标签页
```

## 四、配置体系（init.lua 从0到1）
Neovim的现代配置完全基于Lua，**配置入口文件为 `~/.config/nvim/init.lua`**（Linux/macOS），Windows对应路径为 `%LOCALAPPDATA%\nvim\init.lua`，完全替代传统的 `init.vim`。

### 1. 配置文件目录结构（最佳实践）
模块化配置是长期维护的核心，推荐目录结构如下：
```
~/.config/nvim/
├── init.lua          # 配置入口，负责加载所有模块
├── lua/              # Lua模块目录，所有子配置都放在这里
│   ├── core/         # 核心配置
│   │   ├── options.lua  # 基础选项设置
│   │   ├── keymaps.lua  # 快捷键映射
│   │   └── autocmd.lua  # 自动命令
│   └── plugins/      # 插件配置
│       ├── init.lua  # 插件管理器与插件列表
│       ├── lsp.lua   # LSP相关配置
│       └── treesitter.lua # Tree-sitter配置
└── after/            # 延迟加载的配置（可选）
```

入口文件 `init.lua` 最简示例：
```lua
-- 加载核心配置
require("core.options")
require("core.keymaps")
require("core.autocmd")
-- 加载插件配置
require("plugins.init")
```

### 2. 基础选项配置（options.lua）
通过Lua的vim API设置编辑器行为，替代传统的 `set` 命令，核心API分为三类：
- `vim.o`：全局选项
- `vim.wo`：窗口局部选项
- `vim.bo`：缓冲区局部选项

核心配置示例：
```lua
local opt = vim.opt
local g = vim.g

-- 编码设置
g.encoding = "UTF-8"
opt.fileencoding = "utf-8"

-- 行号设置
opt.number = true         -- 显示行号
opt.relativenumber = true -- 显示相对行号（配合j/k移动更高效）
opt.cursorline = true     -- 高亮当前行
opt.scrolloff = 8         -- 光标上下保留8行，避免滚动到屏幕边缘
opt.sidescrolloff = 8     -- 光标左右保留8列

-- 缩进与Tab设置
opt.tabstop = 4           -- Tab显示为4个空格
opt.shiftwidth = 4        -- 自动缩进的宽度
opt.expandtab = true      -- 将Tab转换为空格
opt.smartindent = true    -- 智能缩进
opt.shiftround = true     -- 缩进对齐到shiftwidth的整数倍

-- 搜索设置
opt.ignorecase = true     -- 搜索忽略大小写
opt.smartcase = true      -- 搜索含大写字母时，自动切换为大小写敏感
opt.hlsearch = true       -- 高亮搜索结果
opt.incsearch = true      -- 增量搜索

-- UI与体验设置
opt.signcolumn = "yes"    -- 始终显示左侧图标列（避免LSP诊断时屏幕抖动）
opt.colorcolumn = "120"   -- 右侧参考线，提示代码长度
opt.termguicolors = true  -- 启用24位真彩色（必备，美化UI的基础）
opt.mouse = "a"           -- 全模式启用鼠标支持
opt.clipboard = "unnamedplus" -- 同步系统剪贴板
opt.updatetime = 300      -- 自动更新的延迟时间（单位ms）
opt.timeoutlen = 500      -- 快捷键组合的超时时间
```

### 3. 快捷键映射（keymaps.lua）
通过 `vim.keymap.set` 定义快捷键，替代传统的 `map` 命令，语法格式：
`vim.keymap.set(模式, 触发键, 执行命令/函数, 配置项)`

模式简写：`n`普通模式，`i`插入模式，`v`可视模式，`t`终端模式，`o`操作符等待模式。

核心映射示例：
```lua
local map = vim.keymap.set
local opt = { noremap = true, silent = true } -- noremap: 非递归映射，silent: 不显示命令行提示

-- 基础优化：取消方向键，强制使用hjkl（新手练习可选）
map("", "<Up>", "<Nop>", opt)
map("", "<Down>", "<Nop>", opt)
map("", "<Left>", "<Nop>", opt)
map("", "<Right>", "<Nop>", opt)

-- Leader键设置（Neovim快捷键的核心前缀，推荐设置为空格）
vim.g.mapleader = " "
vim.g.maplocalleader = " "

-- 窗口切换优化：Ctrl+hjkl直接切换窗口，无需先按Ctrl+w
map("n", "<C-h>", "<C-w>h", opt)
map("n", "<C-j>", "<C-w>j", opt)
map("n", "<C-k>", "<C-w>k", opt)
map("n", "<C-l>", "<C-w>l", opt)

-- 分屏快捷键
map("n", "<leader>sv", ":vs<CR>", opt) -- 垂直分屏
map("n", "<leader>sh", ":sp<CR>", opt) -- 水平分屏
map("n", "<leader>sq", ":q<CR>", opt)  -- 关闭当前窗口

-- 缓冲区切换
map("n", "<leader>bn", ":bn<CR>", opt) -- 下一个缓冲区
map("n", "<leader>bp", ":bp<CR>", opt) -- 上一个缓冲区
map("n", "<leader>bd", ":bd<CR>", opt) -- 关闭当前缓冲区

-- 终端模式优化：Esc退出终端插入模式
map("t", "<Esc>", "<C-\\><C-n>", opt)
```

### 4. 自动命令（autocmd.lua）
自动命令用于在特定事件触发时自动执行操作，比如文件保存时自动格式化、打开特定文件时设置缩进等。

核心示例：
```lua
local autocmd = vim.api.nvim_create_autocmd
local augroup = vim.api.nvim_create_augroup

-- 自定义自动命令组，避免重复定义
local core_group = augroup("CoreAutoCmd", { clear = true })

-- 1. 保存文件时自动去除行尾空格
autocmd("BufWritePre", {
  group = core_group,
  pattern = "*",
  command = "%s/\\s\\+$//e",
})

-- 2. 打开Python文件时，设置缩进为4个空格
autocmd("FileType", {
  group = core_group,
  pattern = "python",
  callback = function()
    vim.opt_local.tabstop = 4
    vim.opt_local.shiftwidth = 4
  end,
})

-- 3. 打开前端文件时，设置缩进为2个空格
autocmd("FileType", {
  group = core_group,
  pattern = { "javascript", "typescript", "html", "css", "vue", "react" },
  callback = function()
    vim.opt_local.tabstop = 2
    vim.opt_local.shiftwidth = 2
  end,
})

-- 4. 打开终端时，自动进入插入模式
autocmd("TermOpen", {
  group = core_group,
  pattern = "*",
  command = "startinsert",
})
```

## 五、内置核心功能详解
### 1. 内置LSP客户端
LSP（Language Server Protocol）是现代编辑器智能补全、代码导航的核心，Neovim原生内置LSP客户端，无需第三方插件即可实现IDE级代码能力。

#### 快速配置（社区标准方案）
社区主流使用 `mason.nvim` + `mason-lspconfig.nvim` + `nvim-lspconfig` 三件套实现LSP的一键安装、管理与配置，无需手动编译安装语言服务器。

1.  安装插件（以lazy.nvim为例）
```lua
-- plugins/init.lua
require("lazy").setup({
  -- LSP核心三件套
  "williamboman/mason.nvim",
  "williamboman/mason-lspconfig.nvim",
  "neovim/nvim-lspconfig",
})
```

2.  基础配置
```lua
-- plugins/lsp.lua
-- 1. 初始化mason
require("mason").setup()
require("mason-lspconfig").setup({
  -- 自动安装的语言服务器
  ensure_installed = {
    "lua_ls",    -- Lua
    "pyright",   -- Python
    "ts_ls",     -- TypeScript/JavaScript
    "gopls",     -- Go
    "rust_analyzer", -- Rust
  },
})

-- 2. 配置LSP能力
local lspconfig = require("lspconfig")
local capabilities = require("cmp_nvim_lsp").default_capabilities() -- 配合nvim-cmp补全

-- 3. 批量配置语言服务器
local servers = { "lua_ls", "pyright", "ts_ls", "gopls", "rust_analyzer" }
for _, server in ipairs(servers) do
  lspconfig[server].setup({
    capabilities = capabilities,
    -- LSP附加配置，绑定快捷键
    on_attach = function(client, bufnr)
      local map = vim.keymap.set
      local opt = { noremap = true, silent = true, buffer = bufnr }
      -- 核心LSP快捷键
      map("n", "gD", vim.lsp.buf.declaration, opt)          -- 跳转到声明
      map("n", "gd", vim.lsp.buf.definition, opt)           -- 跳转到定义
      map("n", "gr", vim.lsp.buf.references, opt)           -- 查看所有引用
      map("n", "K", vim.lsp.buf.hover, opt)                 -- 悬停查看文档
      map("n", "<leader>rn", vim.lsp.buf.rename, opt)       -- 变量重命名
      map("n", "<leader>ca", vim.lsp.buf.code_action, opt)  -- 代码修复/重构
      map("n", "<leader>f", function()                       -- 代码格式化
        vim.lsp.buf.format({ async = true })
      end, opt)
      -- 诊断跳转
      map("n", "[d", vim.diagnostic.goto_prev, opt)         -- 上一个诊断错误
      map("n", "]d", vim.diagnostic.goto_next, opt)         -- 下一个诊断错误
    end,
  })
end

-- Lua专属配置（针对Neovim配置开发）
lspconfig.lua_ls.setup({
  settings = {
    Lua = {
      runtime = { version = "LuaJIT" },
      diagnostics = { globals = { "vim" } }, -- 识别vim全局变量
      workspace = { library = vim.api.nvim_get_runtime_file("", true) },
      telemetry = { enable = false },
    },
  },
})
```

#### 核心LSP命令
| 命令 | 功能 |
| :--- | :--- |
| `:LspInfo` | 查看当前缓冲区LSP服务器运行状态 |
| `:LspInstall` | 安装语言服务器 |
| `:LspUninstall` | 卸载语言服务器 |
| `:Mason` | 打开Mason图形界面，管理LSP/DAP/Linter/Formatter |
| `:lua vim.diagnostic.setloclist()` | 打开诊断列表，查看所有文件错误 |

### 2. Tree-sitter
Tree-sitter是一个增量语法解析器，基于AST抽象语法树实现精准的代码解析，远优于传统正则匹配的语法高亮，同时提供增量选择、代码折叠、智能缩进等能力。

#### 安装与配置
```lua
-- 插件安装
require("lazy").setup({
  {
    "nvim-treesitter/nvim-treesitter",
    build = ":TSUpdate", -- 安装后自动更新解析器
  },
})

-- 基础配置（plugins/treesitter.lua）
require("nvim-treesitter.configs").setup({
  -- 自动安装的语法解析器，"all"安装所有，推荐按需安装
  ensure_installed = {
    "lua", "python", "go", "rust", "javascript", "typescript",
    "html", "css", "json", "yaml", "markdown", "bash", "c", "cpp"
  },
  auto_install = true, -- 打开未知文件类型时，自动安装对应解析器
  highlight = {
    enable = true, -- 启用基于Tree-sitter的语法高亮
    additional_vim_regex_highlighting = false, -- 关闭传统正则高亮
  },
  incremental_selection = {
    enable = true, -- 启用增量选择（按语法节点扩大选择范围）
    keymaps = {
      init_selection = "<CR>", -- 回车开始选择
      node_incremental = "<CR>", -- 回车扩大选择范围
      node_decremental = "<BS>", -- 退格缩小选择范围
    },
  },
  indent = { enable = true }, -- 启用智能缩进
  fold = { enable = true },   -- 启用代码折叠
})

-- 开启基于Tree-sitter的代码折叠
vim.opt.foldmethod = "expr"
vim.opt.foldexpr = "nvim_treesitter#foldexpr()"
vim.opt.foldenable = false -- 默认不折叠
```

### 3. 内置终端模拟器
Neovim原生内置基于libvterm的终端模拟器，无需离开编辑器即可执行shell命令、运行程序，完全兼容Neovim的按键映射与模式。

#### 核心使用方式
| 命令/操作 | 功能 |
| :--- | :--- |
| `:terminal` / `:term` | 新建缓冲区打开终端 |
| `:vs term://bash` | 垂直分屏打开终端 |
| `:sp term://zsh` | 水平分屏打开终端 |
| `i` / `a` | 终端缓冲区进入终端插入模式（可输入shell命令） |
| `Esc` | 终端插入模式返回普通模式（可执行复制、滚动、切换窗口等操作） |
| `Ctrl+\ Ctrl+n` | 强制退出终端插入模式（Esc映射失效时使用） |

#### 常用配置
```lua
-- 快捷键映射：<leader>tt 浮动终端
map("n", "<leader>tt", ":ToggleTerm<CR>", opt) -- 需配合toggleterm.nvim插件，原生可自定义浮动终端
-- 终端模式下，Esc退出插入模式
map("t", "<Esc>", "<C-\\><C-n>", { noremap = true, silent = true })
-- 终端模式下，Ctrl+hjkl切换窗口
map("t", "<C-h>", "<C-\\><C-n><C-w>h", opt)
map("t", "<C-j>", "<C-\\><C-n><C-w>j", opt)
map("t", "<C-k>", "<C-\\><C-n><C-w>k", opt)
map("t", "<C-l>", "<C-\\><C-n><C-w>l", opt)
```

## 六、插件生态与管理
Neovim的强大之处在于繁荣的插件生态，社区已全面转向Lua原生插件，功能覆盖从UI美化到全栈IDE能力的所有场景。

### 1. 主流插件管理器
当前社区首选 **lazy.nvim**，它是现代Neovim插件管理器的标杆，支持异步安装、懒加载、锁文件、图形化管理界面，极致优化启动速度，上百个插件也可实现100ms内启动。

#### lazy.nvim 安装与基础配置
```lua
-- init.lua 开头添加，自动安装lazy.nvim
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- 稳定分支
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

-- 加载lazy.nvim，配置插件列表
require("lazy").setup({
  -- 插件列表，直接写插件GitHub地址简写
  "folke/which-key.nvim", -- 快捷键提示
  "nvim-tree/nvim-tree.lua", -- 文件树
  -- 带配置的插件
  {
    "nvim-telescope/telescope.nvim",
    dependencies = { "nvim-lua/plenary.nvim" }, -- 依赖插件
  },
  -- 更多插件...
}, {
  -- lazy.nvim 全局配置
  install = { colorscheme = { "tokyonight" } }, -- 安装缺失插件时使用的配色
  checker = { enabled = true, notify = false }, -- 自动检查插件更新
})
```

核心命令：
| 命令 | 功能 |
| :--- | :--- |
| `:Lazy` | 打开lazy.nvim图形管理界面 |
| `:Lazy install` | 安装所有缺失插件 |
| `:Lazy update` | 更新所有插件 |
| `:Lazy clean` | 清理未使用的插件 |
| `:Lazy sync` | 同步插件（install+update+clean） |
| `:Lazy check` | 检查插件更新 |

### 2. 核心必备插件推荐
按功能分类，推荐社区主流、稳定维护的Lua原生插件，覆盖绝大多数开发场景：

| 分类 | 插件 | 核心功能 |
| :--- | :--- | :--- |
| **插件管理** | folke/lazy.nvim | 现代插件管理器，异步、懒加载、图形化界面，社区首选 |
| **配色与UI** | folke/tokyonight.nvim | 主流配色方案，支持真彩色，护眼且辨识度高 |
| | catppuccin/nvim | 低对比度柔和配色，多风格可选，适配绝大多数插件 |
| | nvim-lualine/lualine.nvim | 高颜值状态栏，自定义程度高，轻量高效 |
| | folke/noice.nvim | 重构命令行、通知、消息系统，现代化UI体验 |
| **文件管理** | nvim-tree/nvim-tree.lua | 侧边栏文件树，功能全面，轻量稳定 |
| | nvim-neo-tree/neo-tree.nvim | 功能更强大的文件管理器，支持缓冲区、Git、远程文件 |
| **模糊搜索** | nvim-telescope/telescope.nvim | 万能模糊搜索神器，支持文件、文本、符号、Git、缓冲区等全场景搜索 |
| **代码补全** | hrsh7th/nvim-cmp | 社区最强补全引擎，全模块化设计，支持LSP、路径、片段、缓冲区等多种补全源 |
| | L3MON4D3/LuaSnip | 代码片段引擎，配合nvim-cmp实现自定义代码片段 |
| **LSP增强** | williamboman/mason.nvim | LSP/DAP/Linter/Formatter一键安装管理，内置App Store |
| | neovim/nvim-lspconfig | 官方LSP配置合集，提供绝大多数语言服务器的开箱即用配置 |
| | nvimdev/lspsaga.nvim | LSP功能增强，提供更美观的UI、跳转、诊断、代码动作体验 |
| **Git集成** | lewis6991/gitsigns.nvim | 行级Git状态提示，支持 blame、diff、撤销修改等操作 |
| | tpope/vim-fugitive | 全能Git插件，支持所有Git命令，与Neovim深度集成 |
| **终端增强** | akinsho/toggleterm.nvim | 浮动终端、分屏终端管理，支持多终端切换、自定义快捷键 |
| **效率工具** | folke/which-key.nvim | 快捷键实时提示，按下前缀键自动弹出后续按键说明，解决记不住快捷键的痛点 |
| | windwp/nvim-autopairs | 自动补全括号、引号、标签，支持嵌套、多行匹配 |
| | numToStr/Comment.nvim | 极速代码注释，支持单行/多行注释，适配绝大多数语言 |
| **AI辅助** | github/copilot.vim | GitHub Copilot官方插件，AI代码补全 |
| | yetone/avante.nvim | 开源AI编程助手，支持多模型，类似Cursor的对话式编程体验 |

## 七、进阶使用技巧
### 1. 启动性能优化
1.  **懒加载插件**：lazy.nvim原生支持懒加载，通过 `event`、`ft`、`cmd`、`keys` 配置插件触发时机，避免启动时加载所有插件。
    ```lua
    -- 示例：仅在打开Python文件时加载插件
    {
      "linux-cultist/venv-selector.nvim",
      ft = "python", -- 仅Python文件类型触发加载
    }
    -- 示例：仅执行命令时加载
    {
      "akinsho/toggleterm.nvim",
      cmd = "ToggleTerm", -- 执行:ToggleTerm时才加载
      config = true,
    }
    ```
2.  **禁用不必要的内置插件**：Neovim内置了很多兼容Vim的插件，不用的可以禁用，加快启动速度。
    ```lua
    -- init.lua 开头添加
    local builtins = {
      "gzip", "zip", "zipPlugin", "tar", "tarPlugin",
      "getscript", "getscriptPlugin", "vimball", "vimballPlugin",
      "2html_plugin", "logipat", "rrhelper", "spellfile_plugin",
      "matchit", "matchparen", "netrw", "netrwPlugin", "netrwSettings",
    }
    for _, plugin in pairs(builtins) do
      vim.g["loaded_" .. plugin] = 1
    end
    ```
3.  **延迟加载非核心配置**：将非启动必需的配置放到 `VimEnter` 自动命令中，延迟执行。

### 2. 大文件编辑优化
编辑GB级大文件时，禁用重型功能避免卡顿：
```lua
-- 自动命令：大于10MB的文件，禁用Tree-sitter、LSP、语法高亮等
local large_file_group = augroup("LargeFileOpt", { clear = true })
autocmd("BufReadPre", {
  group = large_file_group,
  pattern = "*",
  callback = function()
    local file_size = vim.fn.getfsize(vim.fn.expand("%"))
    if file_size > 10 * 1024 * 1024 then -- 大于10MB
      vim.opt_local.bufhidden = "unload"
      vim.opt_local.buftype = "nowrite"
      vim.opt_local.undolevels = -1 -- 禁用撤销
      vim.opt_local.swapfile = false -- 禁用交换文件
      vim.opt_local.syntax = "off" -- 禁用语法高亮
      vim.cmd("TSBufDisable highlight") -- 禁用Tree-sitter
      vim.diagnostic.disable(0) -- 禁用LSP诊断
    end
  end,
})
```

### 3. 调试集成（DAP）
Neovim通过 `nvim-dap` 实现原生调试能力，支持断点、步进、变量查看、调用栈等全功能调试，兼容VS Code的调试适配器。核心插件：
- mfussenegger/nvim-dap：DAP客户端核心
- rcarriga/nvim-dap-ui：调试UI界面
- theHamsta/nvim-dap-virtual-text：调试变量虚拟文本显示
- jay-babu/mason-nvim-dap.nvim：配合Mason一键安装调试适配器

### 4. 宏录制与批量操作
宏是Neovim的高效批量操作神器，核心用法：
1.  普通模式按 `q+字母` 开始录制宏（如 `qa` 录制到寄存器a）
2.  执行需要重复的操作
3.  按 `q` 结束录制
4.  按 `@+字母` 执行宏（如 `@a` 执行寄存器a的宏），`@@` 重复执行上一次宏
5.  批量执行：`数字+@+字母`，如 `100@a` 执行100次宏

## 八、常见问题与排查
1.  **Neovim启动慢**
    - 执行 `:checkhealth` 检查健康状态，定位报错
    - 执行 `:Lazy profile` 查看插件启动耗时，对耗时高的插件开启懒加载
    - 禁用不必要的内置插件和自动命令

2.  **LSP不生效/不附着**
    - 执行 `:LspInfo` 查看LSP服务器状态，确认是否安装成功
    - 检查文件类型是否匹配LSP配置的 `filetypes`
    - 执行 `:checkhealth lspconfig` 检查LSP配置是否正常
    - 确认语言服务器可在终端正常执行，无环境依赖缺失

3.  **中文乱码**
    - 确认系统终端编码为UTF-8
    - 配置中添加 `vim.o.encoding = "UTF-8"` 和 `vim.o.fileencoding = "utf-8"`
    - 终端字体设置为支持中文的等宽字体（如JetBrains Mono、思源黑体等）

4.  **Tree-sitter高亮不生效**
    - 执行 `:checkhealth nvim-treesitter` 检查解析器是否安装成功
    - 执行 `:TSInstall 语言名` 手动安装对应语言的解析器
    - 确认 `highlight.enable = true` 已开启，且关闭了传统正则高亮

5.  **从Vim迁移到Neovim**
    - Neovim完全兼容Vimscript，可直接将 `.vimrc` 内容复制到 `init.vim` 中使用
    - 建议逐步将Vimscript配置迁移到Lua，享受更好的性能和生态
    - 注意路径变化：Neovim配置目录为 `~/.config/nvim/`，而非 `~/.vim/`

## 九、学习路径与权威资源
1.  **内置帮助文档**：`:help` 是学习Neovim最权威、最全面的资源，任何命令、API、配置都可通过 `:help 关键词` 查看官方文档。
2.  **入门学习**：
    - `vimtutor`：终端执行 `vimtutor`，30分钟掌握Vim/Neovim基础操作
    - Neovim官方文档：https://neovim.io/doc/
3.  **配置参考**：
    - NvChad/LazyVim：开箱即用的Neovim发行版，适合新手直接使用，也可参考其配置结构
    - GitHub上的awesome-neovim：https://github.com/rockerBOO/awesome-neovim，最全的Neovim插件合集
4.  **进阶学习**：
    - Lua官方文档：https://www.lua.org/manual/5.1/ （Neovim使用LuaJIT，兼容Lua 5.1）
    - Neovim Lua API文档：https://neovim.io/doc/user/lua.html
    - 各插件的官方GitHub仓库，查看详细配置与用法示例


