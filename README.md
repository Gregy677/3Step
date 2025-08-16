local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local PlaceID = game.PlaceId
local Cursor = nil

-- Function to get a non-full server
local function getServer()
    local url = ("https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Desc&limit=100%s"):format(
        PlaceID,
        Cursor and ("&cursor=" .. Cursor) or ""
    )

    local success, result = pcall(function()
        return HttpService:JSONDecode(game:HttpGet(url))
    end)

    if success and result and result.data then
        for _, server in ipairs(result.data) do
            if server.id and server.playing < server.maxPlayers then
                return server.id
            end
        end
        Cursor = result.nextPageCursor
    end

    return nil
end

-- Loop: try hopping every 1 second
while task.wait(1) do
    local serverId = getServer()
    if serverId then
        print("Teleporting to server:", serverId)
        TeleportService:TeleportToPlaceInstance(PlaceID, serverId, LocalPlayer)
        break -- stops loop since teleport will reload script in new server
    else
        warn("No available servers found, retrying...")
    end
end