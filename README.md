-- // Settings \\ --

local Settings = {
    Timer = 0.00; -- Time between each shot added
    Amount = 30; -- Amount of shots added
    Transparency = 0.5; -- Chams transparency
    Color = Color3.new(1, 50 / 255, 50 / 255); -- Chams color
}

-- // Variables \\ --

local Fire;
local BodyParts = {}
local Plr = game:GetService("Players").LocalPlayer;

-- // RemoteEvent Hook \\ --

local Hook = newcclosure(function(self, ...)
    if not checkcaller() then
        for i, v in pairs({...}) do
            if BodyParts[tostring(v)] then
                for i = 1, Settings.Amount, 1 do
                    Fire(self, ...);
                    if Settings.Timer > 0 then
                        wait(Settings.Timer)
                    end;
                end
               
            end
        end
    end
    return Fire(self, ...);
end)

Fire = hookfunction(Instance.new("RemoteEvent").FireServer, Hook);

-- // Chams \\ --

local ChamsFolder = Instance.new("Folder", game:GetService("CoreGui"));
local Ignore = {
    ["Particle Area"] = true;
    ["FakeHead"] = true;
    ["HeadHB"] = true;
    ["Hitbox"] = true;
    ["Gun"] = true;
}

SetChams = function(plr)
    if not plr or not plr:IsA("Player") then
        return false;
    end;
    if not ChamsFolder:FindFirstChild(plr.Name) then
        Instance.new("Folder", ChamsFolder).Name = plr.Name;
    end;
    if plr and plr.Character and plr.Character:FindFirstChild("Humanoid") and ChamsFolder:FindFirstChild(plr.Name) then
        ChamsFolder[plr.Name]:ClearAllChildren();
        for i, v in pairs(plr.Character:GetChildren()) do
            if v:IsA("BasePart") and not Ignore[v.Name] then
                if ChamsFolder:FindFirstChild(plr.Name) then
                    local Chams = Instance.new("BoxHandleAdornment", ChamsFolder[plr.Name]);
                    Chams.ZIndex = 3;
                    Chams.AlwaysOnTop = true;
                    Chams.Adornee = v;
                    Chams.Color3 = Settings.Color;
                    Chams.Size = v.Size + Vector3.new(0.1, 0.1, 0.1);
                    Chams.Transparency = Settings.Transparency;
                    Chams.Name = v.Name;
                end;
            end;
            if v:IsA("BasePart") and not BodyParts[v.Name] then
                BodyParts[v.Name] = true;
            end
        end;
    end;
end;

local SetChamsColor = function(self, k)
    if type(self) ~= "userdata" and not self:IsA("Player") and typeof(k) ~= "Color3" then
        return false;
    end;
    if ChamsFolder:FindFirstChild(self.Name) and not (ChamsFolder[self.Name]:FindFirstChildOfClass("BoxHandleAdornment").Color3 == k) then
        for i, v in pairs(ChamsFolder[self.Name]:GetChildren()) do
            if v:IsA("BoxHandleAdornment") then
                v.Color3 = k;
            end;
        end;
    end;
    return true;
end;

for i, v in pairs(game:GetService("Players"):GetPlayers()) do
    if v ~= Plr and v.Team ~= Plr.Team then
        coroutine.wrap(SetChams)(v);
    end;
end;

game:GetService("Players").PlayerAdded:Connect(function(self)
    if self.Team ~= Plr.Team then
        SetChams(self);
    end;
end);

game:GetService("Players").PlayerRemoving:Connect(function(self)
    if ChamsFolder:FindFirstChild(self.Name) then
        pcall(game.Destroy, ChamsFolder[self.Name]);
    end;
end);

coroutine.wrap(function()
    while true do
        for i, v in pairs(game:GetService("Players"):GetPlayers()) do
            if v ~= Plr and v.Team ~= Plr.Team then
                coroutine.wrap(SetChams)(v);
            end;
            if v ~= Plr and v.Team == Plr.Team and ChamsFolder:FindFirstChild(v.Name) then
                pcall(game.Destroy, ChamsFolder[v.Name]);
            end
        end;
        wait(1);
    end;
end)();
