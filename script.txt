-- KAMUS ONLY | EXECUTOR READY
-- Database: words2.txt

local DB_URL = "https://raw.githubusercontent.com/rafaelsakammuhammad-dev/Sambung-Kata/main/words2.txt"
local words = {}
local loaded = false

local function httpGet(url)
    local ok, body = pcall(function()
        if typeof and typeof(game) == "Instance" then
            return game:HttpGet(url)
        end
        if request then
            local r = request({Url = url, Method = "GET"})
            return r.Body
        end
        if http_request then
            local r = http_request({Url = url, Method = "GET"})
            return r.Body
        end
        if syn and syn.request then
            local r = syn.request({Url = url, Method = "GET"})
            return r.Body
        end
        error("Executor tidak menyediakan HTTP request")
    end)
    if ok and type(body) == "string" then return body end
    warn("[KAMUS] Gagal mengambil database: " .. tostring(body))
    return nil
end

local function loadDatabase()
    local data = httpGet(DB_URL)
    if not data or #data == 0 then return false end

    table.clear(words)
    local seen = {}
    for word in data:gmatch("[^\r\n]+") do
        word = word:lower():gsub("^%s+", ""):gsub("%s+$", "")
        if word ~= "" and not word:find("%s") and not seen[word] then
            seen[word] = true
            words[#words + 1] = word
        end
    end

    table.sort(words)
    loaded = true
    print("[KAMUS] words2.txt loaded: " .. #words .. " kata")
    return true
end

local function lowerBound(prefix)
    local lo, hi = 1, #words + 1
    while lo < hi do
        local mid = math.floor((lo + hi) / 2)
        if words[mid] < prefix then
            lo = mid + 1
        else
            hi = mid
        end
    end
    return lo
end

local function findPrefix(prefix, limit)
    prefix = (prefix or ""):lower()
    limit = limit or 200
    if not loaded or prefix == "" then return {} end

    local out = {}
    local index = lowerBound(prefix)
    while index <= #words and #out < limit do
        local word = words[index]
        if word:sub(1, #prefix) ~= prefix then break end
        out[#out + 1] = word
        index += 1
    end
    return out
end

local function makeGui()
    local Players = game:GetService("Players")
    local CoreGui = game:GetService("CoreGui")

    pcall(function()
        local old = CoreGui:FindFirstChild("KamusOnlyExecutor")
        if old then old:Destroy() end
    end)

    local gui = Instance.new("ScreenGui")
    gui.Name = "KamusOnlyExecutor"
    gui.ResetOnSpawn = false
    gui.Parent = CoreGui

    local frame = Instance.new("Frame")
    frame.Size = UDim2.fromOffset(360, 500)
    frame.Position = UDim2.new(0.5, -180, 0.5, -250)
    frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
    frame.BorderSizePixel = 0
    frame.Parent = gui

    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1,0,0,40)
    title.BackgroundTransparency = 1
    title.Text = "KAMUS ONLY | words2"
    title.TextColor3 = Color3.new(1,1,1)
    title.Font = Enum.Font.SourceSansBold
    title.TextSize = 19
    title.Parent = frame

    local input = Instance.new("TextBox")
    input.Size = UDim2.new(1,-20,0,38)
    input.Position = UDim2.fromOffset(10,50)
    input.BackgroundColor3 = Color3.fromRGB(40,40,40)
    input.BorderSizePixel = 0
    input.ClearTextOnFocus = false
    input.PlaceholderText = "Masukkan awalan..."
    input.TextColor3 = Color3.new(1,1,1)
    input.TextSize = 17
    input.Parent = frame
    Instance.new("UICorner", input).CornerRadius = UDim.new(0,6)

    local list = Instance.new("ScrollingFrame")
    list.Size = UDim2.new(1,-20,1,-100)
    list.Position = UDim2.fromOffset(10,98)
    list.BackgroundColor3 = Color3.fromRGB(32,32,32)
    list.BorderSizePixel = 0
    list.ScrollBarThickness = 6
    list.CanvasSize = UDim2.new()
    list.Parent = frame

    local function show(results)
        for _, child in ipairs(list:GetChildren()) do
            if child:IsA("TextLabel") then child:Destroy() end
        end
        for i, word in ipairs(results) do
            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(1,-8,0,24)
            label.Position = UDim2.fromOffset(4,(i-1)*24)
            label.BackgroundTransparency = 1
            label.TextXAlignment = Enum.TextXAlignment.Left
            label.Text = word
            label.TextColor3 = Color3.new(1,1,1)
            label.TextSize = 16
            label.Font = Enum.Font.SourceSans
            label.Parent = list
        end
        list.CanvasSize = UDim2.fromOffset(0,#results*24+8)
    end

    input:GetPropertyChangedSignal("Text"):Connect(function()
        show(findPrefix(input.Text, 200))
    end)
end

if loadDatabase() then
    makeGui()
else
    warn("[KAMUS] Database words2.txt tidak berhasil dimuat.")
end
