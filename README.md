--// Services
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

--// Config
local PlaceID = game.PlaceId
local Cursor = nil
local SeenServers = {}
local MaxAttempts = 1000 -- Pages to check per hop
local HopDelay = 1.2 -- Seconds between hop attempts

-- Load saved server history
local function loadSeenServers()
    pcall(function()
        local data = readfile("SeenServers.json")
        SeenServers = HttpService:JSONDecode(data)
    end)
end

-- Save visited servers
local function saveSeenServers()
    pcall(function()
        writefile("SeenServers.json", HttpService:JSONEncode(SeenServers))
    end)
end

-- Find a new server that is NOT full
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
                -- Only join servers with 1–7 players AND at least 2 empty slots
                if server.playing >= 4 
                and server.playing <= 7 
                and (server.maxPlayers - server.playing) >= 2 
                and not server.vip 
                and not server.privateServerOwnerId 
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

-- Hop to the selected server
local function hopServer()
    local newServerID = findNewServer()
    if newServerID then
        TeleportService:TeleportToPlaceInstance(PlaceID, newServerID, LocalPlayer)
    else
        warn("No new servers found!")
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