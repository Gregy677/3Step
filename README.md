-- ⚡ Unstoppable Fast Server Hopper ⚡
-- Hops every 1 second, avoids full servers, never stops on error

local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

local PlaceId = game.PlaceId
local Player = Players.LocalPlayer
local Cursor = nil

-- settings
local HopDelay = 1.7 -- seconds between hops
local MinSlots = 3 -- how many open slots required

-- Try teleport with error protection
local function SafeTeleport(serverId)
    local ok, err = pcall(function()
        TeleportService:TeleportToPlaceInstance(PlaceId, serverId, Player)
    end)
    if not ok then
        warn("Teleport failed: ".. tostring(err))
    end
end

local function Hop()
    local success, servers = pcall(function()
        local url = "https://games.roblox.com/v1/games/"..PlaceId.."/servers/Public?sortOrder=Asc&limit=100"
        if Cursor then
            url = url.."&cursor="..Cursor
        end
        return HttpService:JSONDecode(game:HttpGet(url))
    end)

    if success and servers and servers.data then
        for _, server in ipairs(servers.data) do
            local maxPlayers = server.maxPlayers
            local playing = server.playing
            local id = server.id

            if maxPlayers - playing >= MinSlots and id ~= game.JobId then
                SafeTeleport(id)
                return
            end
        end
        Cursor = servers.nextPageCursor
    end
end

-- loop hopper forever
task.spawn(function()
    while task.wait(HopDelay) do
        pcall(Hop) -- even if Hop fails, script keeps running
    end
end)