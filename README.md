# flash-zh.nvim

基于[flash.nvim](https://github.com/folke/flash.nvim)以及小鹤双拼，neovim 中文跳转插件。

![iShot_2023-10-05_19 32 53](https://github.com/rainzm/flash-zh.nvim/assets/22927169/4c3ca124-0fee-48a2-b7c6-17391afe8d0e)

## 安装

- 依赖于[flash.nvim](https://github.com/folke/flash.nvim)
- 使用 [lazy.nvim](https://github.com/folke/lazy.nvim) 进行安装:

```lua
return {{
    "rainzm/flash-zh.nvim",
    event = "VeryLazy",
    dependencies = "folke/flash.nvim",
    keys = {{
        "s",
        mode = {"n", "x", "o"},
        function()
            require("flash-zh").jump({
                chinese_only = false
            })
        end,
        desc = "Flash between Chinese"
    }}
}, {
    "folke/flash.nvim",
    event = "VeryLazy",
    opts = {
        highlight = {
            backdrop = false,
            matches = false
        }
    }
}}
```

## 使用

1. ~~label 默认使用大写字母，这样可以避免和拼音冲突。~~ label 现在默认使用小写字母，通过自定义`flash.nvim`的 labeler ，以避免小写 label 和拼音的冲突。
2. 默认工作在中英混杂模式下（由[dirichy](https://github.com/dirichy)实现）；增加选项 `chinese_only` 使其工作在仅中文模式下。
3. `jump`的参数会传递给`flash.nvim`，查看 [issue 2](https://github.com/rainzm/flash-zh.nvim/issues/2) 。

**如果想要跳转的地方没有 label 出现，接着输入即可，和查找一样。**

### Leap 风格标签翻页（Tab / Space 切换下一批）

当屏幕上匹配数很多（例如单字母 `s`→`j` 搜到几十个 j 开头的字）时，标签池会被第一批匹配用完，剩余匹配只会高亮不会显示字母。
flash-zh 现在内置了**按批次滚动标签**的功能，类似 leap.nvim，按翻页键把当前批的字母清空，贴到之前只有高亮的匹配上。

| 按键 | 作用 |
|---|---|
| `<Tab>` / `<C-i>` / `<C-n>` / `<Space>` | 下一批标签 |
| `<S-Tab>` / `<C-p>` | 上一批标签 |

在 `jump()` 调用时通过 flash.nvim 的 `actions` 选项注册即可，记得用 `state:update({ force = true })` 强制刷新（否则 cache 认为 pattern/win 没变会跳过重画）：

```lua
vim.keymap.set({ "n", "x", "o" }, "s", function()
  require("flash-zh").jump({
    chinese_only = false,
    labels = "fjghdkslatuyirewpnvbcmxz;FJGHDLATUYIREQNB,q",
    actions = {
      ["<Tab>"]   = function(s) s._zh_batch = (s._zh_batch or 0) + 1 s:update({ force = true }) return true end,
      ["<C-I>"]   = function(s) s._zh_batch = (s._zh_batch or 0) + 1 s:update({ force = true }) return true end,
      ["<C-n>"]   = function(s) s._zh_batch = (s._zh_batch or 0) + 1 s:update({ force = true }) return true end,
      ["<Space>"] = function(s) s._zh_batch = (s._zh_batch or 0) + 1 s:update({ force = true }) return true end,
      ["<S-Tab>"] = function(s) s._zh_batch = math.max(0, (s._zh_batch or 0) - 1) s:update({ force = true }) return true end,
      ["<C-p>"]   = function(s) s._zh_batch = math.max(0, (s._zh_batch or 0) - 1) s:update({ force = true }) return true end,
    },
  })
end, { desc = "Flash between Chinese (leap-style batch labels)" })
```

> 单字母宽泛搜索时，labeler 不再为所有匹配剔除"拼音续字母"（仅 pattern ≥ 2 字才保留），并限制续字母扫描范围，
> 所以标签池比旧版本更大，再配合分批翻页，基本不需要再靠"继续输拼音"来缩小范围。

### 自定义匹配字符

- 你可以覆盖、或是追加字符到默认的匹配字符集。

    ```lua
    require('flash-zh').setup {
        char_map = {
            -- Override default mapping in `flypy.comma`
            comma = {
                [']'] = ']」', -- A string of chars to match for, with no separator. No need to escape.
                ['!'] = '!！', -- You can add a symbol that isn't present in the default table.
            },
            -- Append to `flypy.comma`
            append_comma = {
                ['.'] = '…',
            },
            -- Append to `flypy.char1patterns`
            append_char1 = {
                ['a'] = 'äÄ',
            },
            -- Append to `flypy.char2patterns`
            append_char2 = {},
        }
    }
    ```

## 感谢

- [hop-zh-by-flypy](https://github.com/zzhirong/hop-zh-by-flypy)

## 推荐

- [rime-ls](https://github.com/wlh320/rime-ls) 通过补全的方式输入中文
