-- โหลดไลบรารี Rayfield (ต้องมีไฟล์ไลบรารีในระบบ)
local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source'))()

-- สร้างหน้าต่างหลัก
local Window = Rayfield:CreateWindow({
    Name = "⚡ Rayfield - ชุดฟังก์ชันปรับแต่ง",
    Icon = 0, -- ใส่ไอคอนที่ต้องการได้
    LoadingTitle = "กำลังโหลดระบบ...",
    LoadingSubtitle = "โดย ผู้พัฒนา",
    Theme = "Default", -- สามารถเปลี่ยนธีมได้ เช่น Dark, Light, Purple

    DisableRayfieldPrompts = false,
    DisableBuildWarnings = false,
    ConfigSaving = {
        Enabled = true,
        FolderName = nil,
        FileName = "RayfieldConfig"
    },

    Discord = {
        Enabled = false,
        Invite = "",
        RememberJoins = true
    }
})

-- 📌 แท็บฟังก์ชันหลัก
local MovementTab = Window:CreateTab("🏃 การเคลื่อนไหว", nil)
local VisualTab = Window:CreateTab("👁️ การมองเห็น", nil)
local MiscTab = Window:CreateTab("💾 อื่นๆ", nil)

-- ==============================================
-- 🏃 ส่วนการเคลื่อนไหว
-- ==============================================
-- วิ่งเร็ว
local SpeedSlider = MovementTab:CreateSlider({
    Name = "⚡ ความเร็วในการวิ่ง",
    Range = {16, 250},
    Increment = 1,
    Suffix = " ความเร็ว",
    CurrentValue = 16,
    Flag = "SpeedValue",
})

MovementTab:CreateToggle({
    Name = "เปิดใช้งานวิ่งเร็ว",
    CurrentValue = false,
    Flag = "SpeedEnabled",
    Callback = function(Value)
        local Player = game.Players.LocalPlayer
        local Character = Player.Character or Player.CharacterAdded:Wait()
        local Humanoid = Character:WaitForChild("Humanoid")
        
        _G.SpeedLoop = Value
        while _G.SpeedLoop do
            task.wait(0.05)
            if Humanoid and Humanoid.Health > 0 then
                Humanoid.WalkSpeed = SpeedSlider.CurrentValue
            end
        end
        if Humanoid then Humanoid.WalkSpeed = 16 end -- คืนค่าเดิมเมื่อปิด
    end,
})

-- บิน
local FlySlider = MovementTab:CreateSlider({
    Name = "🌪️ ความเร็วในการบิน",
    Range = {10, 200},
    Increment = 1,
    Suffix = " ความเร็ว",
    CurrentValue = 25,
    Flag = "FlySpeedValue",
})

MovementTab:CreateToggle({
    Name = "เปิดใช้งานการบิน",
    CurrentValue = false,
    Flag = "FlyEnabled",
    Callback = function(Value)
        local Player = game.Players.LocalPlayer
        local Character = Player.Character or Player.CharacterAdded:Wait()
        local Humanoid = Character:WaitForChild("Humanoid")
        local Root = Character:WaitForChild("HumanoidRootPart")
        
        _G.FlyLoop = Value
        local UIS = game:GetService("UserInputService")
        local ContextAction = game:GetService("ContextActionService")
        
        local function FlyAction(Name, State)
            if State == Enum.UserInputState.Change then
                _G.KeyStates[Name] = (State == Enum.UserInputState.Begin)
            end
        end
        
        _G.KeyStates = {W=false, A=false, S=false, D=false, Space=false, Shift=false}
        ContextAction:BindAction("FlyW", FlyAction, false, Enum.KeyCode.W)
        ContextAction:BindAction("FlyA", FlyAction, false, Enum.KeyCode.A)
        ContextAction:BindAction("FlyS", FlyAction, false, Enum.KeyCode.S)
        ContextAction:BindAction("FlyD", FlyAction, false, Enum.KeyCode.D)
        ContextAction:BindAction("FlyUp", FlyAction, false, Enum.KeyCode.Space)
        ContextAction:BindAction("FlyDown", FlyAction, false, Enum.KeyCode.LeftShift)

        while _G.FlyLoop and task.wait() do
            if Humanoid and Humanoid.Health > 0 and Root then
                local Cam = workspace.CurrentCamera
                local Dir = Vector3.new()
                
                if _G.KeyStates.W then Dir += Cam.CFrame.LookVector end
                if _G.KeyStates.S then Dir -= Cam.CFrame.LookVector end
                if _G.KeyStates.A then Dir -= Cam.CFrame.RightVector end
                if _G.KeyStates.D then Dir += Cam.CFrame.RightVector end
                if _G.KeyStates.Space then Dir += Vector3.new(0,1,0) end
                if _G.KeyStates.Shift then Dir -= Vector3.new(0,1,0) end
                
                if Dir.Magnitude > 0 then Dir = Dir.Unit end
                Root.Velocity = Dir * FlySlider.CurrentValue
                Humanoid.GravityScale = 0
                Humanoid.PlatformStand = true
            end
        end
        
        -- คืนค่าเดิม
        if Humanoid then
            Humanoid.GravityScale = 1
            Humanoid.PlatformStand = false
        end
        ContextAction:UnbindAllActions()
    end,
})

-- กระโดดสูง
local JumpSlider = MovementTab:CreateSlider({
    Name = "🦿 ความสูงกระโดด",
    Range = {5, 150},
    Increment = 1,
    Suffix = " ความสูง",
    CurrentValue = 7.2,
    Flag = "JumpPowerValue",
})

MovementTab:CreateToggle({
    Name = "เปิดใช้งานกระโดดสูง",
    CurrentValue = false,
    Flag = "JumpEnabled",
    Callback = function(Value)
        local Player = game.Players.LocalPlayer
        local Character = Player.Character or Player.CharacterAdded:Wait()
        local Humanoid = Character:WaitForChild("Humanoid")
        
        _G.JumpLoop = Value
        while _G.JumpLoop do
            task.wait(0.1)
            if Humanoid and Humanoid.Health > 0 then
                Humanoid.JumpPower = JumpSlider.CurrentValue
            end
        end
        if Humanoid then Humanoid.JumpPower = 7.2 end
    end,
})

-- ==============================================
-- 👁️ ส่วนการมองเห็นและล็อคเป้า
-- ==============================================
-- มองทะลุสีฟ้า
VisualTab:CreateToggle({
    Name = "👀 มองทะลุวัตถุเป็นสีฟ้า",
    CurrentValue = false,
    Flag = "SeeThroughEnabled",
    Callback = function(Value)
        local Lighting = game:GetService("Lighting")
        if Value then
            Lighting.Ambient = Color3.fromHex("#00ccff")
            Lighting.OutdoorAmbient = Color3.fromHex("#0099ff")
            Lighting.Brightness = 2
        else
            Lighting.Ambient = Color3.fromHex("#888888") -- คืนค่าปกติ
            Lighting.OutdoorAmbient = Color3.fromHex("#1f1f1f")
            Lighting.Brightness = 1
        end
    end,
})

-- เส้นเชื่อมศัตรูสีแดง
VisualTab:CreateToggle({
    Name = "👹 เส้นเชื่อมไปหาคนสีแดง",
    CurrentValue = false,
    Flag = "EnemyLineEnabled",
    Callback = function(Value)
        local RunService = game:GetService("RunService")
        local Player = game.Players.LocalPlayer
        local Camera = workspace.CurrentCamera
        
        _G.LineLoop = Value
        local Lines = {}
        
        while _G.LineLoop and task.wait() do
            -- ลบเส้นเก่า
            for _,v in pairs(Lines) do v:Destroy() end
            table.clear(Lines)
            
            -- วาดเส้นใหม่
            for _,plr in pairs(game.Players:GetPlayers()) do
                if plr ~= Player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    local Root = plr.Character.HumanoidRootPart
                    local Pos, OnScr = Camera:WorldToViewportPoint(Root.Position)
                    
                    if OnScr then
                        local Line = Drawing.new("Line")
                        Line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
                        Line.To = Vector2.new(Pos.X, Pos.Y)
                        Line.Color = Color3.fromHex("#ff0000")
                        Line.Thickness = 1.5
                        Line.Transparency = 1
                        Line.Visible = true
                        table.insert(Lines, Line)
                    end
                end
            end
        end
        -- ล้างเส้นเมื่อปิด
        for _,v in pairs(Lines) do v:Destroy() end
    end,
})

-- ล็อคเป้า
VisualTab:CreateSlider({
    Name = "👾 ระยะล็อคเป้า",
    Range = {50, 500},
    Increment = 10,
    Suffix = " หน่วย",
    CurrentValue = 200,
    Flag = "AimRange",
})

VisualTab:CreateToggle({
    Name = "เปิดใช้งานล็อคเป้า",
    CurrentValue = false,
    Flag = "AimEnabled",
    Callback = function(Value)
        local UIS = game:GetService("UserInputService")
        local Player = game.Players.LocalPlayer
        local Camera = workspace.CurrentCamera
        
        _G.AimLoop = Value
        while _G.AimLoop and task.wait(0.01) do
            if UIS:IsMouseButtonDown(Enum.MouseButton.Right) then -- กดขวาค้างเพื่อล็อค
                local Closest = nil
                local Dist = math.huge
                
                for _,plr in pairs(game.Players:GetPlayers()) do
                    if plr ~= Player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                        local Root = plr.Character.HumanoidRootPart
                        local Pos, OnScr = Camera:WorldToViewportPoint(Root.Position)
                        
                        if OnScr then
                            local Mouse = UIS:GetMouseLocation()
                            local NewDist = (Vector2.new(Pos.X, Pos.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
                            
                            if NewDist < Dist and NewDist < Window.Flags.AimRange then
                                Dist = NewDist
                                Closest = Root
                            end
                        end
                    end
                end
                
                if Closest then
                    Camera.CFrame = CFrame.new(Camera.CFrame.Position, Closest.Position)
                end
            end
        end
    end,
})

-- ==============================================
-- 💾 ส่วนอื่นๆ
-- ==============================================
MiscTab:CreateInput({
    Name = "ใส่รหัสเพลง/ID",
    PlaceholderText = "ใส่เลข ID ที่นี่...",
    RemoveTextAfterFocusLost = false,
    Callback = function(Value)
        -- ตัวอย่างการใช้งาน: เล่นเสียง
        local Sound = Instance.new("Sound")
        Sound.SoundId = "rbxassetid://"..Value
        Sound.Volume = 1
        Sound.Parent = workspace
        Sound:Play()
        game.Debris:AddItem(Sound, 500)
    end,
})

-- แสดงข้อความสำเร็จ
Rayfield:Notify({
    Title = "✅ โหลดสำเร็จ",
    Content = "ระบบพร้อมใช้งานทุกฟังก์ชัน",
    Duration = 3,
    Image = 0
})
