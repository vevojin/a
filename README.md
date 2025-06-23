local HttpService = game:GetService("HttpService")
local MyHRP = game.Players.LocalPlayer.Character:WaitForChild("HumanoidRootPart")
local BallFolder = workspace:WaitForChild("BallFolderServer")

while task.wait(0.25) do
    for _, v in pairs(BallFolder:GetChildren()) do
        local Ball = v:FindFirstChild("Ball")
        local Velocity = v:FindFirstChild("Velocity")
        local BallMagnitude = (v:WaitForChild('Shadow').Position-MyHRP.Position).Magnitude
        local distance = (MyHRP.Position - v.Ball.Position).Magnitude

        if Ball and Velocity then

            local data = {
                player = tostring(MyHRP.Position),
                ball = tostring(Ball.Position),
                velocity = tostring(Velocity.Value)
                speed = Velocity.Value.Magnitude
                isFalling = Velocity.Value.Y < 0
                distance = (MyHRP.Position - Ball.Position).Magnitude
                local sameSide = Ball.Position.Z > 0 == MyHRP.Position.Z > 0
                local timeToLandEstimation = math.abs(Ball.Position.Y / Velocity.Value.Y)
                local relative = MyHRP.CFrame:ToObjectSpace(Ball.CFrame)
            }

            local success, response = pcall(function()
                return request({
                    Url = "http://localhost:8000/decide",
                    Method = "POST",
                    Headers = {
                        ["Content-Type"] = "application/json"
                    },
                    Body = HttpService:JSONEncode(data)
                })
            end)

            if success and response then
                local body = HttpService:JSONDecode(response.Body)
                if body.decision and body.decision ~= "Wait" then
                    print("[AI Dive]:", body.decision)
                end
            end
        end
    end
end
