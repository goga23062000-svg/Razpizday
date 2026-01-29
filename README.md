-- Free Camera Script для Garry's Mod
-- Author: sherik

if CLIENT then
    -- Configuration
    local cameraSpeed = 10
    local fastCameraSpeed = 25
    local mouseSensitivity = 0.5
    
    -- Camera state
    local freeCameraEnabled = false
    local cameraAngles = Angle(0, 0, 0)
    local cameraOrigin = Vector(0, 0, 0)
    local originalViewAngles = Angle(0, 0, 0)
    local originalEyePos = Vector(0, 0, 0)
    
    -- UI Elements
    local toggleBtn
    local statusPanel
    
    -- Colors
    local colors = {
        background = Color(30, 30, 40, 240),
        button = Color(70, 130, 180, 255),
        button_hover = Color(100, 160, 210, 255),
        button_active = Color(50, 180, 100, 255),
        button_active_hover = Color(70, 200, 120, 255),
        text = Color(240, 240, 240, 255),
        text_shadow = Color(20, 20, 20, 150),
        border = Color(60, 60, 80, 255),
        status_bg = Color(40, 40, 50, 220)
    }
    
    -- Create toggle button
    function CreateToggleButton()
        if IsValid(toggleBtn) then toggleBtn:Remove() end
        
        toggleBtn = vgui.Create("DButton")
        toggleBtn:SetSize(140, 40)
        toggleBtn:SetPos(20, 20)
        toggleBtn:SetText("")
        toggleBtn:SetTooltip("Toggle Free Camera\nRight click for menu")
        
        toggleBtn.Paint = function(self, w, h)
            local btnColor
            if freeCameraEnabled then
                btnColor = self:IsHovered() and colors.button_active_hover or colors.button_active
            else
                btnColor = self:IsHovered() and colors.button_hover or colors.button
            end
            
            -- Rounded background
            draw.RoundedBox(8, 0, 0, w, h, btnColor)
            
            -- Border
            draw.RoundedBox(8, 0, 0, w, h, colors.border)
            
            -- Text with shadow
            draw.SimpleText("FREE CAMERA", "DermaDefaultBold", w/2 + 1, h/2 + 1, colors.text_shadow, TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER)
            draw.SimpleText("FREE CAMERA", "DermaDefaultBold", w/2, h/2, colors.text, TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER)
            
            -- Status indicator
            if freeCameraEnabled then
                draw.RoundedBox(4, w - 12, 6, 6, 6, Color(50, 255, 50, 255))
            end
        end
        
        toggleBtn.DoClick = function()
            ToggleFreeCamera()
        end
        
        toggleBtn.DoRightClick = function()
            OpenControlMenu()
        end
    end
    
    -- Create status panel
    function CreateStatusPanel()
        if IsValid(statusPanel) then statusPanel:Remove() end
        
        statusPanel = vgui.Create("DPanel")
        statusPanel:SetSize(300, 120)
        statusPanel:SetPos(ScrW() - 320, 20)
        statusPanel:SetVisible(false)
        
        statusPanel.Paint = function(self, w, h)
            -- Blur background
            Derma_DrawBackgroundBlur(self, CurTime() - (self.CreatedTime or CurTime()))
            
            -- Panel background
            draw.RoundedBox(12, 0, 0, w, h, colors.status_bg)
            
            -- Title
            draw.SimpleText("Free Camera Controls", "DermaLarge", w/2, 15, colors.text, TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER)
            
            -- Controls list
            local controls = {
                {"WASD", "Move camera"},
                {"Q/E", "Move up/down"},
                {"Shift", "Speed boost"},
                {"RMB + Mouse", "Rotate camera"},
                {"Teleport", "Move player to camera"}
            }
            
            for i, control in ipairs(controls) do
                local y = 40 + (i-1) * 16
                draw.SimpleText(control[1], "DermaDefaultBold", 20, y, Color(100, 200, 255), TEXT_ALIGN_LEFT, TEXT_ALIGN_CENTER)
                draw.SimpleText("- " .. control[2], "DermaDefault", 100, y, colors.text, TEXT_ALIGN_LEFT, TEXT_ALIGN_CENTER)
            end
        end
        
        statusPanel.CreatedTime = CurTime()
    end
    
    -- Open control menu
    function OpenControlMenu()
        local menu = DermaMenu()
        
        menu:SetPos(gui.MouseX(), gui.MouseY())
        
        menu:AddOption("Toggle Free Camera", function()
            ToggleFreeCamera()
        end):SetIcon("icon16/camera.png")
        
        menu:AddOption("Teleport to Camera", function()
            TeleportToCamera()
        end):SetIcon("icon16/arrow_right.png")
        
        menu:AddSpacer()
        
        menu:AddOption("Settings", function()
            OpenSettingsMenu()
        end):SetIcon("icon16/cog.png")
        
        menu:AddOption("Help", function()
            ShowHelp()
        end):SetIcon("icon16/help.png")
        
        menu:AddSpacer()
        
        menu:AddOption("Close", function() end):SetIcon("icon16/cross.png")
        
        menu:Open()
    end
    
    -- Open settings menu
    function OpenSettingsMenu()
        local frame = vgui.Create("DFrame")
        frame:SetSize(300, 250)
        frame:SetTitle("Free Camera Settings")
        frame:Center()
        frame:MakePopup()
        
        frame.Paint = function(self, w, h)
            draw.RoundedBox(8, 0, 0, w, h, colors.background)
        end
        
        -- Speed slider
        local speedLabel = vgui.Create("DLabel", frame)
        speedLabel:SetText("Camera Speed: " .. cameraSpeed)
        speedLabel:SetPos(20, 40)
        speedLabel:SizeToContents()
        
        local speedSlider = vgui.Create("DNumSlider", frame)
        speedSlider:SetPos(20, 60)
        speedSlider:SetSize(260, 40)
        speedSlider:SetText("")
        speedSlider:SetMin(1)
        speedSlider:SetMax(50)
        speedSlider:SetDecimals(0)
        speedSlider:SetValue(cameraSpeed)
        speedSlider.OnValueChanged = function(self, value)
            cameraSpeed = value
            speedLabel:SetText("Camera Speed: " .. value)
        end
        
        -- Fast speed slider
        local fastLabel = vgui.Create("DLabel", frame)
        fastLabel:SetText("Fast Speed: " .. fastCameraSpeed)
        fastLabel:SetPos(20, 110)
        fastLabel:SizeToContents()
        
        local fastSlider = vgui.Create("DNumSlider", frame)
        fastSlider:SetPos(20, 130)
        fastSlider:SetSize(260, 40)
        fastSlider:SetText("")
        fastSlider:SetMin(10)
        fastSlider:SetMax(100)
        fastSlider:SetDecimals(0)
        fastSlider:SetValue(fastCameraSpeed)
        fastSlider.OnValueChanged = function(self, value)
            fastCameraSpeed = value
            fastLabel:SetText("Fast Speed: " .. value)
        end
        
        -- Sensitivity slider
        local sensLabel = vgui.Create("DLabel", frame)
        sensLabel:SetText("Mouse Sensitivity: " .. mouseSensitivity)
        sensLabel:SetPos(20, 180)
        sensLabel:SizeToContents()
        
        local sensSlider = vgui.Create("DNumSlider", frame)
        sensSlider:SetPos(20, 200)
        sensSlider:SetSize(260, 40)
        sensSlider:SetText("")
        sensSlider:SetMin(0.1)
        sensSlider:SetMax(3)
        sensSlider:SetDecimals(1)
        sensSlider:SetValue(mouseSensitivity)
        sensSlider.OnValueChanged = function(self, value)
            mouseSensitivity = value
            sensLabel:SetText("Mouse Sensitivity: " .. value)
        end
        
        -- Reset button
        local resetBtn = vgui.Create("DButton", frame)
        resetBtn:SetText("Reset to Default")
        resetBtn:SetSize(260, 30)
        resetBtn:SetPos(20, 210)
        resetBtn.DoClick = function()
            cameraSpeed = 10
            fastCameraSpeed = 25
            mouseSensitivity = 0.5
            speedSlider:SetValue(cameraSpeed)
            fastSlider:SetValue(fastCameraSpeed)
            sensSlider:SetValue(mouseSensitivity)
        end
    end
    
    -- Show help
    function ShowHelp()
        local helpText = [[
Free Camera System

Controls:
• WASD - Move camera
• Q/E - Move up/down
• Shift - Speed boost
• RMB + Mouse - Rotate camera
• Right-click button - Open menu

Features:
• Fly anywhere in the map
• Teleport player to camera position
• Adjustable settings
• Minimal UI

Press the button again to exit free camera mode.]]
        
        Derma_Message(helpText, "Free Camera Help", "OK")
    end
    
    -- Toggle free camera
    function ToggleFreeCamera()
        freeCameraEnabled = not freeCameraEnabled
        
        if freeCameraEnabled then
            -- Save original data
            local ply = LocalPlayer()
            originalViewAngles = ply:EyeAngles()
            originalEyePos = ply:EyePos()
            
            -- Set initial camera position
            cameraAngles = originalViewAngles
            cameraOrigin = originalEyePos
            
            -- Hide player HUD
            ply:ConCommand("cl_drawhud 0")
            ply:ConCommand("r_drawviewmodel 0")
            
            -- Show mouse cursor
            gui.EnableScreenClicker(true)
            
            -- Update button appearance
            if IsValid(toggleBtn) then
                toggleBtn:SetTooltip("Click to exit free camera")
            end
            
            -- Show controls panel
            if IsValid(statusPanel) then
                statusPanel:SetVisible(true)
            end
            
            chat.AddText(Color(100, 255, 100), "[FreeCamera] ", Color(255, 255, 255), "Enabled! Use WASD to move, RMB to look around.")
        else
            -- Restore original view
            local ply = LocalPlayer()
            ply:ConCommand("cl_drawhud 1")
            ply:ConCommand("r_drawviewmodel 1")
            gui.EnableScreenClicker(false)
            
            -- Update button appearance
            if IsValid(toggleBtn) then
                toggleBtn:SetTooltip("Toggle Free Camera\nRight click for menu")
            end
            
            -- Hide controls panel
            if IsValid(statusPanel) then
                statusPanel:SetVisible(false)
            end
            
            chat.AddText(Color(255, 100, 100), "[FreeCamera] ", Color(255, 255, 255), "Disabled.")
        end
    end
    
    -- Teleport to camera position
    function TeleportToCamera()
        if not freeCameraEnabled then
            chat.AddText(Color(255, 200, 50), "[FreeCamera] ", Color(255, 255, 255), "Enable free camera first!")
            return
        end
        
        net.Start("FreeCamera_Teleport")
        net.WriteVector(cameraOrigin)
        net.SendToServer()
        
        chat.AddText(Color(100, 200, 255), "[FreeCamera] ", Color(255, 255, 255), "Teleported to camera position!")
    end
    
    -- Camera movement
    function MoveCamera()
        if not freeCameraEnabled then return end
        
        local frameTime = FrameTime()
        local speed = input.IsKeyDown(KEY_LSHIFT) and fastCameraSpeed or cameraSpeed
        local move = Vector(0, 0, 0)
        
        -- Forward/backward
        if input.IsKeyDown(KEY_W) then
            move = move + cameraAngles:Forward()
        elseif input.IsKeyDown(KEY_S) then
            move = move - cameraAngles:Forward()
        end
        
        -- Left/right
        if input.IsKeyDown(KEY_D) then
            move = move + cameraAngles:Right()
        elseif input.IsKeyDown(KEY_A) then
            move = move - cameraAngles:Right()
        end
        
        -- Up/down
        if input.IsKeyDown(KEY_E) then
            move = move + Vector(0, 0, 1)
        elseif input.IsKeyDown(KEY_Q) then
            move = move - Vector(0, 0, 1)
        end
        
        -- Normalize and apply speed
        if move:Length() > 0 then
            move:Normalize()
            cameraOrigin = cameraOrigin + move * speed * frameTime
        end
    end
    
    -- Camera rotation
    function RotateCamera()
        if not freeCameraEnabled then return end
        if not input.IsMouseDown(MOUSE_RIGHT) then return end
        
        local mouseX = input.GetCursorPos()
        local mouseY = input.GetCursorPos()
        
        local centerX = ScrW() / 2
        local centerY = ScrH() / 2
        input.SetCursorPos(centerX, centerY)
        
        local deltaX = (mouseX - centerX) * mouseSensitivity * 0.05
        local deltaY = (mouseY - centerY) * mouseSensitivity * 0.05
        
        cameraAngles.p = math.Clamp(cameraAngles.p - deltaY, -89, 89)
        cameraAngles.y = cameraAngles.y + deltaX
    end
    
    -- Update camera position
    hook.Add("Think", "FreeCamera_Think", function()
        if not freeCameraEnabled then return end
        
        MoveCamera()
        RotateCamera()
    end)
    
    -- Override camera view
    hook.Add("CalcView", "FreeCamera_CalcView", function(ply, origin, angles, fov)
        if not freeCameraEnabled then return end
        
        local view = {}
        view.origin = cameraOrigin
        view.angles = cameraAngles
        view.fov = fov
        view.drawviewer = true
        
        return view
    end)
    
    -- Hide HUD elements
    hook.Add("HUDShouldDraw", "FreeCamera_HideHUD", function(name)
        if freeCameraEnabled then
            if name == "CHudHealth" or 
               name == "CHudBattery" or 
               name == "CHudAmmo" or 
               name == "CHudSecondaryAmmo" or
               name == "CHudCrosshair" then
                return false
            end
        end
    end)
    
    -- Initialize UI
    hook.Add("InitPostEntity", "FreeCamera_InitUI", function()
        timer.Simple(1, function()
            CreateToggleButton()
            CreateStatusPanel()
            
            print("[FreeCamera] System loaded!")
            print("[FreeCamera] Click the button in top-left corner to toggle free camera")
            print("[FreeCamera] Right-click the button for menu")
        end)
    end)
    
    -- Console commands
    concommand.Add("freecam_toggle", ToggleFreeCamera)
    concommand.Add("freecam_teleport", TeleportToCamera)
    concommand.Add("freecam_menu", OpenControlMenu)
    
else
    -- Server-side teleportation
    util.AddNetworkString("FreeCamera_Teleport")
    
    net.Receive("FreeCamera_Teleport", function(len, ply)
        if not IsValid(ply) then return end
        
        local targetPos = net.ReadVector()
        
        if targetPos and targetPos:Length() > 0 then
            -- Check if position is valid
            local trace = util.TraceLine({
                start = targetPos + Vector(0, 0, 10),
                endpos = targetPos - Vector(0, 0, 100),
                filter = ply
            })
            
            if trace.Hit then
                ply:SetPos(trace.HitPos + Vector(0, 0, 5))
            else
                ply:SetPos(targetPos)
            end
            
            ply:EmitSound("ambient/energy/zap" .. math.random(1, 3) .. ".wav")
        end
    end)
end
