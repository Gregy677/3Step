-- ⚡ Ultra Fast Server Hopper ⚡
-- Prioritizes servers with more open slots
-- Retries quickly if teleport fails
-- Hops every 1s without stopping

local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

local PlaceId = game.PlaceId
local Player = Players.LocalPlayer
local Cursor = nil

-- settings
local HopDelay = 1 -- seconds between hop checks
local MinSlots = 3 -- minimum open slots required

-- Teleport safely (never stops on error)
local function SafeTeleport(serverId)
    local ok, err = pcall(function()
        TeleportService:TeleportToPlaceInstance(PlaceId, serverId, Player)
    end)
    if not ok then
        warn("Teleport failed: ".. tostring(err))
        return false
    end
    return true
end

-- Grab server list and pick best one
local function GetServers()
    local success, servers = pcall(function()
        local url = "https://games.roblox.com/v1/games/"..PlaceId.."/servers/Public?sortOrder=Asc&limit=100"
        if Cursor then
            url = url.."&cursor="..Cursor
        end
        return HttpService:JSONDecode(game:HttpGet(url))
    end)

    if success and servers and servers.data then
        Cursor = servers.nextPageCursor
        return servers.data
    end
    return {}
end

-- Main hopper
local function Hop()
    local servers = GetServers()
    if #servers == 0 then return end

    -- Sort by MOST open slots
    table.sort(servers, function(a, b)
        return (a.maxPlayers - a.playing) > (b.maxPlayers - b.playing)
    end)

    -- Try servers one by one until teleport succeeds
    for _, server in ipairs(servers) do
        local openSlots = server.maxPlayers - server.playing
        if openSlots >= MinSlots and server.id ~= game.JobId then
            if SafeTeleport(server.id) then
                return
            end
        end
    end
end

-- loop forever
task.spawn(function()
    while task.wait(HopDelay) do
        pcall(Hop)
    end
end)