--// SUNDA HUB //--

local SETTINGS = {
    Keywords = {"marshmello", "water", "gelatin", "sugar"},
    Delay = 1.5,
    Teleport = true,
    UsePrompt = true,

    SellerPosition = CFrame.new(0,5,0), -- UBAH
    AutoSell = false,
    SellDelay = 10,

    AutoBuy = false,
    BuyDelay = 15
}

local running = true

-- UI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Frame = Instance.new("Frame", ScreenGui)
local Title = Instance.new("TextLabel", Frame)
local Status = Instance.new("TextLabel", Frame)
local Info = Instance.new("TextLabel", Frame)

Frame.Size = UDim2.new(0, 240, 0, 180)
Frame.Position = UDim2.new(0.05, 0, 0.3, 0)
Frame.BackgroundColor3 = Color3.fromRGB(20,20,20)

Title.Size = UDim2.new(1,0,0.2,0)
Title.Text = "SUNDA HUB"
Title.TextColor3 = Color3.fromRGB(0,255,127)
Title.BackgroundTransparency = 1

Status.Size = UDim2.new(1,0,0.2,0)
Status.Position = UDim2.new(0,0,0.2,0)
Status.Text = "Farm: ON"
Status.TextColor3 = Color3.fromRGB(255,255,255)
Status.BackgroundTransparency = 1

Info.Size = UDim2.new(1,0,0.2,0)
Info.Position = UDim2.new(0,0,0.4,0)
Info.Text = "[CTRL] Farm | [T] Seller | [Y] Sell | [U] Buy"
Info.TextColor3 = Color3.fromRGB(200,200,200)
Info.BackgroundTransparency = 1

-- ICON
local function createIcon(id, x)
    local icon = Instance.new("ImageLabel", Frame)
    icon.Size = UDim2.new(0,40,0,40)
    icon.Position = UDim2.new(0,x,0,110)
    icon.BackgroundTransparency = 1
    icon.Image = "rbxassetid://"..id
end

createIcon(6031075938, 10)
createIcon(6031091002, 70)
createIcon(6031071053, 130)

-- GET HRP
local function getHRP()
    local p = game.Players.LocalPlayer
    if p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
        return p.Character.HumanoidRootPart
    end
end

-- TELEPORT
local function goSeller()
    local hrp = getHRP()
    if hrp then
        hrp.CFrame = SETTINGS.SellerPosition
        task.wait(1)
    end
end

-- AUTO SELL
local function sellItems()
    goSeller()

    for _, v in pairs(workspace:GetDescendants()) do
        if string.find(string.lower(v.Name), "sell") then
            for _, p in pairs(v:GetDescendants()) do
                if p:IsA("ProximityPrompt") then
                    fireproximityprompt(p)
                    task.wait(0.5)
                end
            end
        end
    end
end

-- AUTO BUY
local function buyItems()
    goSeller()

    for _, v in pairs(workspace:GetDescendants()) do
        if string.find(string.lower(v.Name), "shop") 
        or string.find(string.lower(v.Name), "buy") then
            
            for _, p in pairs(v:GetDescendants()) do
                if p:IsA("ProximityPrompt") then
                    fireproximityprompt(p)
                    print("Beli bahan...")
                    task.wait(0.5)
                end
            end
        end
    end
end

-- CONTROL
game:GetService("UserInputService").InputBegan:Connect(function(input, gp)
    if gp then return end

    if input.KeyCode == Enum.KeyCode.RightControl then
        running = not running
        Status.Text = "Farm: " .. (running and "ON" or "OFF")
    end

    if input.KeyCode == Enum.KeyCode.T then
        goSeller()
    end

    if input.KeyCode == Enum.KeyCode.Y then
        sellItems()
    end

    if input.KeyCode == Enum.KeyCode.U then
        buyItems()
    end
end)

-- FARM
local function isTarget(name)
    name = string.lower(name)
    for _, k in pairs(SETTINGS.Keywords) do
        if string.find(name, k) then
            return true
        end
    end
    return false
end

local function findTargets()
    local t = {}
    for _, v in pairs(workspace:GetDescendants()) do
        if isTarget(v.Name) then
            table.insert(t, v)
        end
    end
    return t
end

task.spawn(function()
    while true do
        task.wait(SETTINGS.Delay)

        if not running then continue end

        local hrp = getHRP()
        if not hrp then continue end

        for _, obj in pairs(findTargets()) do
            if obj:IsA("BasePart") then
                hrp.CFrame = obj.CFrame + Vector3.new(0,3,0)
                task.wait(0.3)
            end

            for _, v in pairs(obj:GetDescendants()) do
                if v:IsA("ProximityPrompt") then
                    fireproximityprompt(v)
                end
            end
        end
    end
end)

-- LOOP AUTO SELL
task.spawn(function()
    while true do
        task.wait(SETTINGS.SellDelay)
        if SETTINGS.AutoSell then
            sellItems()
        end
    end
end)

-- LOOP AUTO BUY
task.spawn(function()
    while true do
        task.wait(SETTINGS.BuyDelay)
        if SETTINGS.AutoBuy then
            buyItems()
        end
    end
end)
