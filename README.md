-- ⚡ Fast Server Hopper ⚡
-- Hops every 1 second and avoids full servers

local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

local PlaceId = game.PlaceId
local Player = Players.LocalPlayer
local Cursor = nil

-- settings
local HopDelay = 1.5 -- seconds between hops
local MinSlots = 1 -- how many open slots required

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
                TeleportService:TeleportToPlaceInstance(PlaceId, id, Player)
                return
            end
        end
        Cursor = servers.nextPageCursor
    end
end

-- loop hopper
task.spawn(function()
    while task.wait(HopDelay) do
        Hop()
    end
end)