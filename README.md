local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local PlaceID = game.PlaceId
local Cursor = nil
local SeenServers = {}
local MaxAttempts = 100
local HopDelay = 1.5
local MinPlayers, MaxPlayers = 2, 8

-- Load saved servers
local function loadSeenServers()
    if isfile and isfile("SeenServers.json") then
        pcall(function()
            local data = readfile("SeenServers.json")
            SeenServers = HttpService:JSONDecode(data)
        end)
    end
end

-- Save seen servers
local function saveSeenServers()
    if writefile then
        pcall(function()
            writefile("SeenServers.json", HttpService:JSONEncode(SeenServers))
        end)
    end
end

-- Find a new server
local function findNewServer()
    local attempts = 0
    while attempts < MaxAttempts do
        local url = ("https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100%s"):format(
            PlaceID,
            Cursor and ("&cursor=" .. Cursor) or ""
        )

        local success, result = pcall(function()
            return HttpService:JSONDecode(game:HttpGet(url))
        end)

        if success and result and result.data then
            for _, server in ipairs(result.data) do
                if server.id
                   and server.playing >= MinPlayers
                   and server.playing <= MaxPlayers
                   and (server.maxPlayers - server.playing) > 0
                   and not SeenServers[server.id] then

                    SeenServers[server.id] = true
                    saveSeenServers()
                    return server.id
                end
            end
            Cursor = result.nextPageCursor
            if not Cursor then break end
        else
            break
        end
        attempts += 1
    end
    return nil
end

-- Handle teleport failures
TeleportService.TeleportInitFailed:Connect(function(player, teleportResult, errorMessage)
    warn("Teleport failed:", teleportResult, errorMessage)
    task.wait(5) -- wait a bit before retry
    -- Try hopping again
    task.spawn(function()
        local id = findNewServer()
        if id then
            TeleportService:TeleportToPlaceInstance(PlaceID, id, LocalPlayer)
        end
    end)
end)

-- Teleport logic
local hopping = false
local function hopServer()
    if hopping then return end
    hopping = true
    local newServerID = findNewServer()
    if newServerID then
        local success, err = pcall(function()
            TeleportService:TeleportToPlaceInstance(PlaceID, newServerID, LocalPlayer)
        end)
        if not success then
            warn("Teleport error:", err)
            hopping = false
        end
    else
        warn("No new servers found! Retrying later...")
        hopping = false
        task.wait(10)
    end
end

-- Start hopper loop
loadSeenServers()
task.spawn(function()
    while true do
        task.wait(HopDelay)
        hopServer()
    end
end)