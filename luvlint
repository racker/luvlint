#!/usr/bin/env lua

for i,v in ipairs(arg) do
  if string.match(v, '.+%.lua') then
    filename = v
  end
end

LUA_PATH = filename

local lxsh = require('lxsh')

--local test_module = assert(require(filename))

local file = assert(io.open(filename, 'r'))

local content = file:read('*all')

assert(file:close())

local parsed = {}
parsed.type = {}
parsed.text = {}
parsed.line = {}
parsed.word = {}

local lines = {}

local line_index = 1

local instance_index = 1

for kind, text, lnum, cnum in lxsh.lexers.lua.gmatch(content, {join_identifiers = true}) do
  table.insert(parsed.type, kind)
  table.insert(parsed.text, text)
  table.insert(parsed.line, lnum)
  table.insert(parsed.word, cnum)
end

while parsed.text[instance_index] do
  lines[line_index] = ''
  while parsed.line[instance_index] == line_index do
    lines[line_index] = lines[line_index]..parsed.text[instance_index]
    instance_index = instance_index + 1
  end
  line_index = line_index + 1
end
 
assert(parsed, 'Parsing Unsuccessful')

local function globalCheck()
  local nativeVars = {"LUA_PATH", "_G", "_LOADED", "_TRACEBACK", "_VERSION", "__pow", "arg", "assert", "collectgarbage", "coroutine", "debug", "dofile", "error",
  "gcinfo", "getfenv", "getmetatable", "io", "ipairs", "loadfile",
  "loadlib", "loadstring", "math", "newproxy", "next", "os", "pairs",
  "pcall", "print", "rawequal", "rawget", "rawset", "require",
  "setfenv", "setmetatable", "string", "table", "tonumber", "tostring",
  "type", "unpack", "xpcall"}

  local localVars = {}

  local globalVars = {}
  
  local index = 1
  
  while parsed.text[index] do
    if parsed.type[index] == 'identifier' then
      globalVars[tostring(parsed.line[index])..':'..tostring(parsed.word[index])] = parsed.text[index]
      if string.match(lines[parsed.line[index]], 'local %S+ ') then
        var = string.gsub(string.match(lines[parsed.line[index]],'local %S+ '), 'local ', '')
        table.insert(localVars, var)
      end
    end
    index = index + 1
  end
  
  print(table.concat(localVars, ' '))
  
  for k,v in pairs(globalVars) do
    for ii, vv in ipairs(localVars) do
      if string.match(v, '^'..vv..'%.') or v == vv then
      	k = nil
      	v = nil
      	break
      end
    end
  end

  for k,v in pairs(globalVars) do
    for iii, vvv in ipairs(nativeVars) do
      if string.match(v, '^'..vvv..'%.') or v == vvv then
      	k = nil
      	v = nil
      	break
      end
    end
  end
  
  for k, v in pairs(globalVars) do
    print('WARNING: '..v..' at ('..k..') is defined as GLOBAL')
  end
end
      	
local function requireCheck()
  local index = 0
  
  local deps = {}
  
  while true do
    index = index + 1
    if parsed.type[index] == 'string' and parsed.text[index - 2] == 'require' then
      deps[tostring(parsed.line[index - 6])..':'..tostring(parsed.word[index - 6])] = parsed.text[index - 6]
    elseif parsed.type[index] == nil then
      break
    end
  end
  
  for k,v in pairs(deps) do
    if type(test_module.v) ~= 'table' then
      print('WARNING: '..v..' at ('..k..') is an imported module that is not a table')
    end
  end
end

for i, v in ipairs(arg) do
  if v == '-g' then
    local x = globalCheck()
  elseif v == '-r' then
    local y = requireCheck()
  end
end
