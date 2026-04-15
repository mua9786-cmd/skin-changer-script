-- // ====== OPTIMIZED VERSION ====== //

if _G.optimizedScriptLoaded then return end
_G.optimizedScriptLoaded = true

local Players = game:GetService("Players")
local ReplicatedFirst = game:GetService("ReplicatedFirst")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer

print("optimized bypass started")

-- // ===== Fake Remote ===== //
local fake = Instance.new("RemoteEvent")
fake.Name = "ClientAlert"
fake.Parent = player

-- // ===== Hook (최소화) ===== //
local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt, false)

mt.__namecall = newcclosure(function(self, ...)
    local method = getnamecallmethod()

    if method == "Kick" then return end
    if method == "FireServer" and self == fake then return end

    return old(self, ...)
end)

setreadonly(mt, true)

-- // ===== AntiCheat 제거 (가벼운 방식) ===== //
local ls3 = ReplicatedFirst:FindFirstChild("LocalScript3")
if ls3 then
    print("AC detected (skipped heavy bypass)")
end

-- // ===== Modules ===== //
local controllers = player.PlayerScripts:WaitForChild("Controllers")
local EnumLibrary = require(ReplicatedStorage.Modules:WaitForChild("EnumLibrary"))
local CosmeticLibrary = require(ReplicatedStorage.Modules:WaitForChild("CosmeticLibrary"))
local ItemLibrary = require(ReplicatedStorage.Modules:WaitForChild("ItemLibrary"))
local DataController = require(controllers:WaitForChild("PlayerDataController"))

if EnumLibrary then EnumLibrary:WaitForEnumBuilder() end

-- // ===== Data ===== //
local equipped, favorites = {}, {}
local lastUsedWeapon
local saveFile = "unlockall/config.json"

-- // ===== Cache ===== //
local cosmeticCache = {}

local function cloneCosmetic(name)
    if cosmeticCache[name] then
        return table.clone(cosmeticCache[name])
    end

    local base = CosmeticLibrary.Cosmetics[name]
    if not base then return end

    local data = {}
    for k, v in pairs(base) do data[k] = v end
    data.Name = name

    cosmeticCache[name] = data
    return table.clone(data)
end

-- // ===== Save (디바운스) ===== //
local saving = false
local function saveConfig()
    if not writefile or saving then return end
    saving = true

    task.delay(1, function()
        pcall(function()
            local config = {equipped = equipped, favorites = favorites}
            makefolder("unlockall")
            writefile(saveFile, HttpService:JSONEncode(config))
        end)
        saving = false
    end)
end

-- // ===== Load ===== //
local function loadConfig()
    if not readfile or not isfile or not isfile(saveFile) then return end
    pcall(function()
        local config = HttpService:JSONDecode(readfile(saveFile))
        equipped = config.equipped or {}
        favorites = config.favorites or {}
    end)
end

-- // ===== Unlock ===== //
CosmeticLibrary.OwnsCosmetic = function()
    return true
end

-- // ===== DataController Hook ===== //
local oldGet = DataController.Get
DataController.Get = function(self, key)
    local data = oldGet(self, key)

    if key == "CosmeticInventory" then
        return setmetatable({}, {__index = function() return true end})
    end

    if key == "FavoritedCosmetics" then
        return favorites
    end

    return data
end

-- // ===== Equip Hook ===== //
local remotes = ReplicatedStorage:FindFirstChild("Remotes")
if remotes then
    local equipRemote = remotes:FindFirstChild("Data") and remotes.Data:FindFirstChild("EquipCosmetic")

    if equipRemote and hookmetamethod then
        local oldNamecall
        oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
            if getnamecallmethod() ~= "FireServer" then
                return oldNamecall(self, ...)
            end

            local args = {...}

            if self == equipRemote then
                local weapon, type, name = args[1], args[2], args[3]

                equipped[weapon] = equipped[weapon] or {}

                if not name or name == "None" then
                    equipped[weapon][type] = nil
                else
                    equipped[weapon][type] = cloneCosmetic(name)
                end

                saveConfig()
                return
            end

            return oldNamecall(self, ...)
        end)
    end
end

-- // ===== ViewModel ===== //
local ClientItem
pcall(function()
    ClientItem = require(player.PlayerScripts.Modules.ClientReplicatedClasses.ClientFighter.ClientItem)
end)

if ClientItem and ClientItem._CreateViewModel then
    local old = ClientItem._CreateViewModel

    ClientItem._CreateViewModel = function(self, vm)
        local weapon = self.Name

        if equipped[weapon] and equipped[weapon].Skin and vm then
            if vm.Data then
                vm.Data.Skin = equipped[weapon].Skin
                vm.Data.Name = equipped[weapon].Skin.Name
            end
        end

        return old(self, vm)
    end
end

-- // ===== Load ===== //
loadConfig()

-- // ===== Auto Reinject ===== //
player.CharacterAdded:Connect(function()
    task.wait(2)
    _G.optimizedScriptLoaded = nil
    loadstring(game:HttpGet("PASTE_YOUR_SCRIPT_URL_HERE"))()
end)
