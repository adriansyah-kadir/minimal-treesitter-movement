this is a minimal treesitter movement in neovim similiar to helix treesitter movement
- '\<a-o>' to increment node
- '\<a-i>' to decrement node
- '\<a-n>' to next node
- '\<a-p>' to prev node

```lua
ts_utils = require('nvim-treesitter.ts_utils')
local m = {}

local function init()
  if m["base"] == nil or vim.fn.mode() == 'n' then
    m["base"] = ts_utils.get_node_at_cursor()
    m["node"] = m["base"]
    return m['node'] ~= nil and 'ok',true or 'failed',false
  end
  return "exists", true
end

local function increment_node()
  local state,status = init()

  if state == 'exists' then
    local parent = m['node']:parent()
    if parent == nil then return end

    while is_node_equal(parent, m['node']) do
      parent = parent:parent()
      if parent == nil then return end
    end

    m['node'] = parent
  end

  if status then
    ts_utils.update_selection(0, m['node'])
  end
end

local function decrement_node()
  local state,status = init()

  if state == 'exists' then
    ::A::
    local child_n = m['node']:child_count()
    if child_n == 0 then return end
    if child_n == 1 then
      local child = m['node']:child(0)
      if is_node_equal(child, m['node']) then
	m['node'] = child
	goto A
      end
      m['node'] = child
    end
    if child_n > 1 then
      for i = 0, child_n do
	local child = m['node']:child(i)
	if ts_utils.is_parent(child, m['base']) then
	  m['node'] = child
	  goto E
	end
      end
      m['node'] = m['node']:child(0)
    end
  end

  ::E::
  if status then
    ts_utils.update_selection(0, m['node'])
  end
end

local function next_node()
  local state,ok = init()

  if state == 'exists' then
    ::A::
    local next = m['node']:next_sibling()
    if next == nil then
      local parent = m['node']:parent()
      if parent == nil then return end
      if is_node_equal(parent, m['node']) then
	m['node'] = parent
	goto A
      end
      next = parent
    end
    m['node'] = next
  end

  if ok then
    ts_utils.update_selection(0, m['node'])
  end
end

local function prev_node()
  local state,ok = init()

  if state == 'exists' then
    ::A::
    local next = m['node']:prev_sibling()
    if next == nil then
      local parent = m['node']:parent()
      if parent == nil then return end
      if is_node_equal(parent, m['node']) then
	m['node'] = parent
	goto A
      end
      next = parent
    end
    m['node'] = next
  end

  if ok then
    ts_utils.update_selection(0, m['node'])
  end
end

vim.keymap.set('n', '<a-o>', increment_node)
vim.keymap.set('x', '<a-o>', increment_node)

vim.keymap.set('n', '<a-i>', decrement_node)
vim.keymap.set('x', '<a-i>', decrement_node)

vim.keymap.set('n', '<a-n>', next_node)
vim.keymap.set('x', '<a-n>', next_node)

vim.keymap.set('n', '<a-p>', prev_node)
vim.keymap.set('x', '<a-p>', prev_node)
```
