local HttpService = game:GetService("HttpService")
local Bots = {"TOKEN1", "TOKEN2", "TOKEN3"} -- Insira os tokens das contas
local Webhook = "https://seuwebhook.com" -- Canal de comunicação dos bots

function SendToCloud(data)
    local payload = HttpService:JSONEncode(data)
    syn.request({
        Url = Webhook,
        Method = "POST",
        Headers = {["Content-Type"] = "application/json"},
        Body = payload
    })
end

function ScanServer()
    local fruits, bosses, players = {}, {}, {}
    for _,obj in pairs(game.Workspace:GetDescendants()) do
        if string.find(obj.Name, "Fruit") then
            table.insert(fruits, obj.Position)
        elseif string.find(obj.Name, "Boss") then
            table.insert(bosses, obj.Position)
        elseif obj:IsA("Model") and game.Players:GetPlayerFromCharacter(obj) then
            table.insert(players, obj.Name)
        end
    end

    SendToCloud({
        ServerID = game.JobId,
        Fruits = fruits,
        Bosses = bosses,
        Players = players,
        Status = "Online"
    })
end

while true do
    ScanServer()
    wait(10) -- Atualiza a nuvem a cada 10 segundos
endfunction CheckCloud()
    local res = syn.request({
        Url = Webhook,
        Method = "GET"
    })
    local data = HttpService:JSONDecode(res.Body)

    for _,server in pairs(data) do
        if #server.Fruits > 0 then
            game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, server.ServerID)
            AI.Speak("Fruta rara detectada no servidor. Teleportando...")
        elseif #server.Bosses > 0 then
            game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, server.ServerID)
            AI.Speak("Boss detectado no servidor. Movendo para combate...")
        end
    end
end

while true do
    CheckCloud()
    wait(15)
end
