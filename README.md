(function()
    -- Serviços principais (não ofuscados)
    local HttpService = game:GetService("HttpService")
    local TeleportService = game:GetService("TeleportService")
    local TweenService = game:GetService("TweenService")
    local Players = game:GetService("Players")
    
    -- URLs montadas com table.concat
    local url_api1 = table.concat({
        "https://", "yoidkwhatsthis", ".elijahmoses-j", ".workers.dev",
        "/api/", "messages"
    })
    
    local url_api2 = table.concat({
        "https://", "brainrot-finder", "-default-rtdb", ".firebaseio.com",
        "/brainrots/", "latest.json"
    })
    
    -- Configurações
    local cfg_gameid = 109983668079237
    local cfg_lifetime = 15  -- Reduzido de 35 para 15 segundos
    local cfg_maxcards = 3   -- Reduzido de 5 para 3 cards
    local cfg_interval1 = 6  -- Aumentado de 4 para 6 segundos
    local cfg_interval2 = 2  -- Aumentado de 1.5 para 2 segundos
    
    -- Variáveis globais (ofuscação leve)
    local ui_gui, ui_frame, tbl_cards, tbl_seen, last_id, is_busy = nil, nil, {}, {}, nil, false
    
    -- Detecção de método HTTP para API 1 (melhorada para PC)
    local function detect_http_method1()
        local methods = {
            -- Método 1: game:HttpGet (mais compatível com PC)
            function(url)
                return game:HttpGet(url)
            end,
            -- Método 2: HttpService:GetAsync (PC padrão)
            function(url)
                return HttpService:GetAsync(url)
            end,
            -- Método 3: syn.request (exploits)
            function(url)
                if syn and syn.request then
                    local r = syn.request({Url=url, Method="GET"})
                    if r.Success and r.StatusCode == 200 then
                        return r.Body
                    end
                end
                return nil
            end,
            -- Método 4: request (exploits)
            function(url)
                if request then
                    local r = request({Url=url, Method="GET"})
                    if r.Success and r.StatusCode == 200 then
                        return r.Body
                    end
                end
                return nil
            end,
            -- Método 5: http_request (exploits)
            function(url)
                if http_request then
                    local r = http_request({Url=url, Method="GET"})
                    if r.Success and r.StatusCode == 200 then
                        return r.Body
                    end
                end
                return nil
            end,
            -- Método 6: http.request (exploits)
            function(url)
                if http and http.request then
                    local r = http.request({Url=url, Method="GET"})
                    if r.Success and r.StatusCode == 200 then
                        return r.Body
                    end
                end
                return nil
            end
        }
        
        -- Testa cada método na API 1
        for idx, method in ipairs(methods) do
            local ok, result = pcall(function()
                return method(url_api1)
            end)
            
            if ok and result and result ~= "null" and result ~= "" and #result > 10 then
                print("✓ API 1 usando método", idx)
                return method
            else
                if not ok then
                    warn("API1 método", idx, "falhou:", result)
                end
            end
        end
        
        warn("⚠ API 1: Nenhum método HTTP funcionou para PC/Mobile")
        return nil
    end
    
    -- Detecção de método HTTP para API 2 (otimizado para PC)
    local function detect_http_method2()
        local methods = {
            -- Método 1: game:HttpGet (melhor para PC)
            function(url)
                return game:HttpGet(url)
            end,
            -- Método 2: HttpService:GetAsync (padrão PC)
            function(url)
                return HttpService:GetAsync(url)
            end,
            -- Método 3: syn.request (exploits)
            function(url)
                if syn and syn.request then
                    local r = syn.request({Url=url, Method="GET"})
                    if r.Success and r.StatusCode == 200 then
                        return r.Body
                    end
                end
                return nil
            end,
            -- Método 4: request (exploits)
            function(url)
                if request then
                    local r = request({Url=url, Method="GET"})
                    if r.Success and r.StatusCode == 200 then
                        return r.Body
                    end
                end
                return nil
            end,
            -- Método 5: http_request (exploits)
            function(url)
                if http_request then
                    local r = http_request({Url=url, Method="GET"})
                    if r.Success and r.StatusCode == 200 then
                        return r.Body
                    end
                end
                return nil
            end
        }
        
        for idx, method in ipairs(methods) do
            local ok, result = pcall(function()
                return method(url_api2)
            end)
            
            if ok and result and result ~= "null" and result ~= "" then
                print("✓ API 2 usando método", idx)
                return method
            else
                if not ok then
                    warn("API2 método", idx, "falhou:", result)
                end
            end
        end
        
        warn("⚠ API 2: Nenhum método HTTP funcionou")
        return nil
    end
    
    local http_method1 = detect_http_method1()
    local http_method2 = detect_http_method2()
    
    -- Funções utilitárias
    local function format_number(num)
        if num >= 1000000000 then
            return string.format("%.2fB", num / 1000000000)
        elseif num >= 1000000 then
            return string.format("%.2fM", num / 1000000)
        elseif num >= 1000 then
            return string.format("%.1fK", num / 1000)
        else
            return tostring(math.floor(num))
        end
    end
    
    local function parse_money_value(text)
        if not text then return 0 end
        
        text = tostring(text):gsub("%*", ""):gsub("$", ""):gsub(",", ""):gsub(" ", "")
        local number = text:match("([%d%.]+)")
        if not number then return 0 end
        
        number = tonumber(number) or 0
        local suffix = text:match("[kKmMbB]")
        
        if suffix then
            suffix = suffix:lower()
            if suffix == "k" then return number * 1000
            elseif suffix == "m" then return number * 1000000
            elseif suffix == "b" then return number * 1000000000
            end
        end
        
        return number
    end
    
    local function is_valid_jobid(job_id)
        if not job_id or type(job_id) ~= "string" then 
            warn("JobID inválido: não é string ou é nil")
            return false 
        end
        
        job_id = job_id:match("^%s*(.-)%s*$")
        
        if #job_id < 10 then 
            warn("JobID inválido: muito curto (", #job_id, "caracteres)")
            return false 
        end
        
        if job_id == "Unknown" then 
            warn("JobID inválido: 'Unknown'")
            return false 
        end
        
        if not job_id:match("^[%w%-]+$") then 
            warn("JobID inválido: caracteres inválidos")
            return false 
        end
        
        return true
    end
    
    -- API 1: Discord Webhook (melhorada para PC)
    local function fetch_api1_data()
        if not http_method1 then 
            warn("⚠ API1: Método HTTP não disponível")
            return nil 
        end
        
        local ok, result = pcall(function()
            return http_method1(url_api1)
        end)
        
        if not ok then
            warn("❌ Erro ao buscar API1:", result)
            return nil
        end
        
        if not result or result == "" or result == "null" then
            warn("⚠ API1: Resposta vazia")
            return nil
        end
        
        -- Verifica se é JSON válido
        if not result:match("^%s*%[") and not result:match("^%s*{") then
            warn("⚠ API1: Resposta não é JSON:", result:sub(1, 100))
            return nil
        end
        
        local decode_ok, data = pcall(function()
            return HttpService:JSONDecode(result)
        end)
        
        if not decode_ok then
            warn("❌ Erro ao decodificar API1:", data)
            return nil
        end
        
        if not data then
            warn("⚠ API1: Dados vazios após decode")
            return nil
        end
        
        return data
    end
    
    local function parse_discord_embed(embed)
        local info = {
            name = "Unknown",
            money = "Unknown",
            job = "Unknown",
            moneyVal = 0,
            source = "API1"
        }
        
        if not embed or not embed.fields then return info end
        
        for _, field in pairs(embed.fields) do
            local field_name = field.name:lower()
            local field_value = field.value or ""
            
            if field_name:find("name") or field_name:find("🏷️") then
                info.name = field_value:gsub("%*", ""):match("^%s*(.-)%s*$")
            elseif field_name:find("money") or field_name:find("💰") then
                info.money = field_value:gsub("%*", ""):match("^%s*(.-)%s*$")
                info.moneyVal = parse_money_value(field_value)
            elseif field_name:find("job") or field_name:find("🆔") then
                info.job = field_value:gsub("```", ""):gsub("`", ""):match("^%s*(.-)%s*$")
            end
        end
        
        return info
    end
    
    -- API 2: Firebase
    local function fetch_api2_data()
        if not http_method2 then return nil end
        
        local ok, result = pcall(function()
            return http_method2(url_api2)
        end)
        
        if not ok then
            warn("Erro ao buscar API2:", result)
            return nil
        end
        
        if not result or result == "null" or result == "" then
            return nil
        end
        
        local decode_ok, data = pcall(function()
            return HttpService:JSONDecode(result)
        end)
        
        if not decode_ok then
            warn("Erro ao decodificar API2:", data)
            return nil
        end
        
        if not data then return nil end
        
        if data.name and data.generation then
            local brainrot_id = data.name .. tostring(data.generation) .. (data.jobId or "")
            
            if brainrot_id ~= last_id then
                last_id = brainrot_id
                
                local gen_text = tostring(data.generation)
                local money_val = parse_money_value(gen_text)
                
                return {
                    name = data.name,
                    money = gen_text,
                    job = data.jobId or "Unknown",
                    moneyVal = money_val,
                    source = "API2"
                }
            end
        end
        
        return nil
    end
    
    -- Sistema de teleporte (NÃO OFUSCAR TeleportService)
    local function teleport_to_server(job_id, card)
        if not is_valid_jobid(job_id) then
            if card then
                TweenService:Create(card, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(150, 30, 30)}):Play()
                task.wait(0.6)
                TweenService:Create(card, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(20, 20, 20)}):Play()
            end
            warn("⚠ JobID inválido para teleporte:", job_id)
            return false
        end
        
        if card then
            TweenService:Create(card, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(255, 215, 0)}):Play()
        end
        
        print("🚀 Iniciando teleporte para JobID:", job_id)
        
        local ok, err = pcall(function()
            TeleportService:TeleportToPlaceInstance(cfg_gameid, job_id, Players.LocalPlayer)
        end)
        
        if not ok then
            warn("✗ ERRO NO TELEPORTE:", err)
            if card then
                task.wait(0.5)
                TweenService:Create(card, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(150, 30, 30)}):Play()
            end
            return false
        end
        
        print("✓ Teleporte iniciado com sucesso!")
        return true
    end
    
    -- Gerenciamento de cards
    local function remove_card(card)
        if not card or not card.Parent then return end
        
        for _, child in pairs(card:GetChildren()) do
            if child:IsA("TextLabel") then
                TweenService:Create(child, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {TextTransparency = 1}):Play()
            elseif child:IsA("UIStroke") then
                TweenService:Create(child, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Transparency = 1}):Play()
            end
        end
        
        TweenService:Create(card, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {BackgroundTransparency = 1}):Play()
        
        task.wait(0.25)
        
        for idx, card_data in ipairs(tbl_cards) do
            if card_data.card == card then
                table.remove(tbl_cards, idx)
                break
            end
        end
        
        card:Destroy()
        
        task.wait(0.05)
        for idx, card_data in ipairs(tbl_cards) do
            local target_pos = UDim2.new(0, 10, 0, (idx-1) * 65 + 10)
            TweenService:Create(
                card_data.card, 
                TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
                {Position = target_pos}
            ):Play()
        end
    end
    
    local function remove_oldest_card()
        if #tbl_cards > 0 then
            remove_card(tbl_cards[1].card)
            task.wait(0.3)
        end
    end
    
    local function create_card(server_data)
        while is_busy do
            task.wait(0.1)
        end
        
        is_busy = true
        
        if #tbl_cards >= cfg_maxcards then
            remove_oldest_card()
        end
        
        local card_idx = #tbl_cards + 1
        
        local card = Instance.new("Frame")
        card.Name = "BrainrotCard_" .. server_data.source
        card.Size = UDim2.new(1, -20, 0, 60)
        card.Position = UDim2.new(0, 10, 0, (card_idx-1) * 65 + 10)
        card.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        card.BorderSizePixel = 0
        card.BackgroundTransparency = 1
        card.Parent = ui_frame
        
        local card_corner = Instance.new("UICorner", card)
        card_corner.CornerRadius = UDim.new(0, 10)
        
        local border_color = server_data.source == "API1" 
            and Color3.fromRGB(255, 215, 0)
            or Color3.fromRGB(138, 43, 226)
        
        local stroke = Instance.new("UIStroke", card)
        stroke.Color = border_color
        stroke.Thickness = 2
        stroke.Transparency = 1
        
        local source_label = Instance.new("TextLabel", card)
        source_label.Size = UDim2.new(0, 55, 0, 16)
        source_label.Position = UDim2.new(0, 8, 0, 4)
        source_label.BackgroundTransparency = 1
        source_label.Text = server_data.source
        source_label.TextColor3 = border_color
        source_label.Font = Enum.Font.GothamBold
        source_label.TextSize = 11
        source_label.TextXAlignment = Enum.TextXAlignment.Left
        source_label.TextTransparency = 1
        
        local name_label = Instance.new("TextLabel", card)
        name_label.Size = UDim2.new(0.55, -15, 0, 24)
        name_label.Position = UDim2.new(0, 8, 0, 24)
        name_label.BackgroundTransparency = 1
        name_label.Text = server_data.name
        name_label.TextColor3 = Color3.fromRGB(255, 255, 255)
        name_label.Font = Enum.Font.GothamBold
        name_label.TextSize = 15
        name_label.TextXAlignment = Enum.TextXAlignment.Left
        name_label.TextTruncate = Enum.TextTruncate.AtEnd
        name_label.TextTransparency = 1
        
        local display_money = server_data.moneyVal > 0 
            and format_number(server_data.moneyVal) 
            or server_data.money
        
        local money_label = Instance.new("TextLabel", card)
        money_label.Size = UDim2.new(0.45, -15, 0, 24)
        money_label.Position = UDim2.new(0.55, 0, 0, 24)
        money_label.BackgroundTransparency = 1
        money_label.Text = display_money
        money_label.TextColor3 = border_color
        money_label.Font = Enum.Font.GothamBold
        money_label.TextSize = 16
        money_label.TextXAlignment = Enum.TextXAlignment.Right
        money_label.TextTransparency = 1
        
        local click_button = Instance.new("TextButton", card)
        click_button.Size = UDim2.new(1, 0, 1, 0)
        click_button.BackgroundTransparency = 1
        click_button.Text = ""
        click_button.ZIndex = 2
        
        local card_data = {
            card = card,
            jobId = server_data.job,
            createdAt = tick(),
            source = server_data.source
        }
        table.insert(tbl_cards, card_data)
        
        TweenService:Create(
            card, 
            TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), 
            {BackgroundTransparency = 0}
        ):Play()
        
        TweenService:Create(
            stroke, 
            TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
            {Transparency = 0}
        ):Play()
        
        TweenService:Create(
            source_label, 
            TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
            {TextTransparency = 0}
        ):Play()
        
        TweenService:Create(
            name_label, 
            TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
            {TextTransparency = 0}
        ):Play()
        
        TweenService:Create(
            money_label, 
            TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
            {TextTransparency = 0}
        ):Play()
        
        click_button.MouseButton1Click:Connect(function()
            teleport_to_server(server_data.job, card)
        end)
        
        click_button.MouseEnter:Connect(function()
            TweenService:Create(
                card, 
                TweenInfo.new(0.2, Enum.EasingStyle.Quad), 
                {BackgroundColor3 = Color3.fromRGB(35, 35, 35)}
            ):Play()
        end)
        
        click_button.MouseLeave:Connect(function()
            TweenService:Create(
                card, 
                TweenInfo.new(0.2, Enum.EasingStyle.Quad), 
                {BackgroundColor3 = Color3.fromRGB(20, 20, 20)}
            ):Play()
        end)
        
        task.spawn(function()
            task.wait(cfg_lifetime)
            remove_card(card)
        end)
        
        is_busy = false
    end
    
    local function process_new_server(server_data)
        if not server_data or not server_data.job then return end
        if not is_valid_jobid(server_data.job) then return end
        if tbl_seen[server_data.job] then return end
        
        tbl_seen[server_data.job] = true
        create_card(server_data)
        
        local short_jobid = server_data.job:sub(1, 15) .. "..."
        print(string.format("📢 [%s] %s | %s | %s", 
            server_data.source, 
            server_data.name, 
            server_data.money, 
            short_jobid
        ))
    end
    
    local function create_ui()
        local player = Players.LocalPlayer
        if not player then return end
        
        local old_ui = player.PlayerGui:FindFirstChild("DualAPIBrainrots")
        if old_ui then old_ui:Destroy() end
        
        ui_gui = Instance.new("ScreenGui")
        ui_gui.Name = "DualAPIBrainrots"
        ui_gui.ResetOnSpawn = false
        ui_gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        ui_gui.Parent = player.PlayerGui
        
        ui_frame = Instance.new("Frame")
        ui_frame.Size = UDim2.new(0, 450, 0, 350)
        ui_frame.Position = UDim2.new(0.5, -225, 0, 15)
        ui_frame.BackgroundTransparency = 1
        ui_frame.BorderSizePixel = 0
        ui_frame.Parent = ui_gui
    end
    
    local function monitor_api1()
        local error_count = 0
        local max_errors = 3  -- Reduzido para reconectar mais rápido
        local consecutive_empty = 0
        local max_empty = 10
        
        while true do
            if http_method1 then
                local data = fetch_api1_data()
                
                if data and type(data) == "table" then
                    error_count = 0
                    consecutive_empty = 0
                    
                    local found_valid = false
                    for _, message in pairs(data) do
                        if message.embeds and #message.embeds > 0 then
                            local server_data = parse_discord_embed(message.embeds[1])
                            if server_data.name ~= "Unknown" and server_data.job ~= "Unknown" then
                                process_new_server(server_data)
                                found_valid = true
                                break
                            end
                        end
                    end
                    
                    if not found_valid then
                        consecutive_empty = consecutive_empty + 1
                        if consecutive_empty >= max_empty then
                            warn("⚠ API 1: Muitas respostas vazias, testando métodos novamente...")
                            http_method1 = detect_http_method1()
                            consecutive_empty = 0
                        end
                    end
                else
                    error_count = error_count + 1
                    if error_count >= max_errors then
                        warn("⚠ API 1: Reconectando... (tentativa", error_count, ")")
                        http_method1 = detect_http_method1()
                        error_count = 0
                    end
                end
            else
                warn("⚠ API 1: Tentando detectar método HTTP...")
                http_method1 = detect_http_method1()
            end
            
            task.wait(cfg_interval1)
        end
    end
    
    local function monitor_api2()
        local error_count = 0
        local max_errors = 10
        
        while true do
            if http_method2 then
                local server_data = fetch_api2_data()
                
                if server_data then
                    error_count = 0
                    process_new_server(server_data)
                else
                    error_count = error_count + 1
                    if error_count >= max_errors then
                        warn("⚠ API 2: Reconectando...")
                        http_method2 = detect_http_method2()
                        error_count = 0
                    end
                end
            end
            
            task.wait(cfg_interval2)
        end
    end
    
    local function initialize()
        local player = Players.LocalPlayer
        if not player then
            repeat task.wait(0.5) until Players.LocalPlayer
            player = Players.LocalPlayer
        end
        
        print("═══════════════════════════════════════")
        print("🔥 DUAL API BRAINROT FINDER v2.2")
        print("   Compatível com PC e Mobile")
        print("═══════════════════════════════════════")
        
        -- Testa APIs antes de iniciar
        print("🔍 Testando compatibilidade das APIs...")
        
        if http_method1 then
            print("📡 API 1 (Discord): ✓ ATIVO")
            -- Teste rápido
            local test_data = fetch_api1_data()
            if test_data then
                print("   ✓ Conexão API 1 confirmada!")
            else
                warn("   ⚠ API 1 conectada mas sem dados no momento")
            end
        else
            warn("📡 API 1 (Discord): ✗ INATIVO")
            warn("   ⚠ Nenhum método HTTP funcionou para API 1")
            warn("   ⚠ Apenas API 2 (Firebase) funcionará")
        end
        
        if http_method2 then
            print("📡 API 2 (Firebase): ✓ ATIVO")
            local test_data = fetch_api2_data()
            if test_data then
                print("   ✓ Conexão API 2 confirmada!")
            else
                print("   ⚠ API 2 conectada mas sem dados no momento")
            end
        else
            warn("📡 API 2 (Firebase): ✗ INATIVO")
        end
        
        print("⏱️  Intervalo API 1:", cfg_interval1, "segundos")
        print("⏱️  Intervalo API 2:", cfg_interval2, "segundos")
        print("📦 Cards máximos:", cfg_maxcards)
        print("⏳ Duração cards:", cfg_lifetime, "segundos")
        print("🎮 Game ID:", cfg_gameid)
        print("═══════════════════════════════════════")
        
        if not http_method1 and not http_method2 then
            warn("❌ ERRO CRÍTICO: Nenhuma API funcionou!")
            warn("⚠️  Verifique sua conexão com a internet")
            warn("⚠️  Ou tente usar outro executor")
            return
        end
        
        create_ui()
        
        -- Inicia monitores
        if http_method1 then
            task.spawn(monitor_api1)
            print("✅ Monitor API 1 iniciado")
        end
        
        if http_method2 then
            task.spawn(monitor_api2)
            print("✅ Monitor API 2 iniciado")
        end
        
        print("✅ Sistema iniciado com sucesso!")
        print("🎯 Aguardando brainrots...")
        print("💡 Cards amarelos = API 1 (Discord)")
        print("💜 Cards roxos = API 2 (Firebase)")
    end
    
    initialize()
end)()

-- Parte 2: Botões UI com funcionalidades
local plr_service = game:GetService("Players")
local uis_service = game:GetService("UserInputService")
local local_plr = plr_service.LocalPlayer

local btn_gui = Instance.new("ScreenGui")
btn_gui.Name = table.concat({"Quadrados", "Premium"})
btn_gui.Parent = local_plr:WaitForChild("PlayerGui")

-- Estados dos scripts
local farm_active = false
local esp_active = false
local farm_thread = nil
local esp_thread = nil

-- Script 1: Farm automatizado (ofuscado)
local function start_farm()
    if farm_active then return end
    farm_active = true
    
    farm_thread = task.spawn(function()
        local cframe_list = {
            Vector3.new(-478, 14, 219),
            Vector3.new(-478, 13, 113),
            Vector3.new(-478, 14, 6),
            Vector3.new(-478, 14, -102),
            Vector3.new(-341, 13, -99),
            Vector3.new(-341, 14, 8),
            Vector3.new(-341, 13, 114),
            Vector3.new(-341, 14, 221),
        }
        
        local plrs = game:GetService('Players')
        local rs = game:GetService('ReplicatedStorage')
        local p = plrs.LocalPlayer
        
        local function get_char_parts()
            local chr = p.Character or p.CharacterAdded:Wait()
            local root = chr:WaitForChild('HumanoidRootPart', 2)
            local hum = chr:FindFirstChildOfClass('Humanoid')
            return chr, hum, root
        end
        
        local c, h, hrp = get_char_parts()
        
        local coil_name = 'Coil Combo'
        local grapple_name, carpet_name = 'Grapple Hook', 'Flying Carpet'
        local remote_path = rs:WaitForChild('Packages'):WaitForChild('Net'):WaitForChild('RE/UseItem')
        local args_tbl = {0.28693917592366536}
        local ascend_h = 25
        
        local function equip_tool(tool_name)
            c, h, hrp = get_char_parts()
            local t = p.Backpack:FindFirstChild(tool_name) or c:FindFirstChild(tool_name)
            if t then
                h:EquipTool(t)
                return t
            end
            return nil
        end
        
        local function parse_gen_num(txt)
            txt = txt:gsub('[%$%s]', '')
            local num_part, sfx = txt:match('([%d%.]+)([KMB]?)')
            num_part = tonumber(num_part) or 0
            local mult = (sfx == 'K' and 1e3) or (sfx == 'M' and 1e6) or (sfx == 'B' and 1e9) or 1
            return num_part * mult
        end
        
        local function find_highest_gen()
            local best_val = -math.huge
            local best_info = nil
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj:IsA('TextLabel') and obj.Name == 'Generation' then
                    if not string.find(obj.Text:lower(), 'fusing') then
                        local val = parse_gen_num(obj.Text)
                        if val > best_val then
                            local disp_obj = obj.Parent:FindFirstChild('DisplayName')
                            local rare_obj = obj.Parent:FindFirstChild('Rarity')
                            if not (disp_obj and string.find(disp_obj.Text:lower(), 'fusing')) then
                                local par, mdl = obj.Parent, nil
                                while par and par ~= workspace do
                                    if par:IsA('Model') and par.Parent and par.Parent.Name == 'Plots' then
                                        mdl = par
                                        break
                                    end
                                    par = par.Parent
                                end
                                
                                local prim_part = mdl and (mdl.PrimaryPart or mdl:FindFirstChildWhichIsA('BasePart'))
                                best_val = val
                                best_info = {
                                    displayName = disp_obj and disp_obj.Text or 'N/A',
                                    generation = obj.Text,
                                    rarity = rare_obj and rare_obj.Text or 'N/A',
                                    value = val,
                                    model = mdl,
                                    primaryPart = prim_part,
                                }
                            end
                        end
                    end
                end
            end
            return best_info
        end
        
        local function find_nearest_cf(pos)
            local closest, dist = nil, math.huge
            for _, v in ipairs(cframe_list) do
                local d = (Vector3.new(v.X, pos.Y, v.Z) - pos).Magnitude
                if d < dist then
                    dist = d
                    closest = v
                end
            end
            return closest
        end
        
        local function tp_sequence(tp_pos)
            c, h, hrp = get_char_parts()
            if not hrp then return end
            
            local coil = equip_tool(coil_name)
            if coil then
                hrp.AssemblyLinearVelocity = Vector3.new(0, 75, 0)
            end
            
            local target_y = hrp.Position.Y + ascend_h
            repeat task.wait() until hrp.Position.Y >= target_y
            
            hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            hrp.CFrame = CFrame.new(tp_pos)
            
            local g = equip_tool(grapple_name)
            if g then
                pcall(function()
                    remote_path:FireServer(unpack(args_tbl))
                end)
            end
            
            task.wait(0.05)
            
            local f = equip_tool(carpet_name)
            if f then
                pcall(function()
                    f:Activate()
                end)
            end
        end
        
        print('🔍 Farm iniciado...')
        while farm_active do
            local highest = find_highest_gen()
            if highest and highest.primaryPart then
                local brain_pos = highest.primaryPart.Position
                local nearest = find_nearest_cf(brain_pos)
                if nearest then
                    local tp_pos = Vector3.new(nearest.X, brain_pos.Y + 2, nearest.Z)
                    tp_sequence(tp_pos)
                end
            end
            task.wait(2)
        end
    end)
end

local function stop_farm()
    farm_active = false
    if farm_thread then
        task.cancel(farm_thread)
        farm_thread = nil
    end
    print('❌ Farm desativado')
end

-- Script 2: ESP visual (ofuscado)
local function start_esp()
    if esp_active then return end
    esp_active = true
    
    esp_thread = task.spawn(function()
        local plrs = game:GetService("Players")
        local ws = game:GetService("Workspace")
        local lp = plrs.LocalPlayer
        
        local esp_folder = Instance.new("Folder")
        esp_folder.Name = "ESP"
        esp_folder.Parent = lp:WaitForChild("PlayerGui")
        
        local line_folder = Instance.new("Folder")
        line_folder.Name = "ESP_Lines"
        line_folder.Parent = ws
        
        local cur_esp, cur_beam, cur_pet_info, a0, a1
        
        local function parse_mps(txt)
            txt = txt:match("^%s*(.-)%s*$")
            local num, sfx = txt:match("%$([%d%.]+)%s*([kKmMbB]?)%s*/?s?")
            num = tonumber(num) or 0
            if sfx then
                sfx = sfx:lower()
                if sfx == "k" then num = num * 1000
                elseif sfx == "m" then num = num * 1_000_000
                elseif sfx == "b" then num = num * 1_000_000_000 end
            end
            return num
        end
        
        local function create_esp(pet_name, info_txt, mut_txt, target_part)
            if cur_esp then cur_esp:Destroy() end
            
            local bb = Instance.new("BillboardGui")
            bb.Size = UDim2.new(0, 150, 0, 50)
            bb.AlwaysOnTop = true
            bb.Adornee = target_part
            bb.Parent = esp_folder
            bb.StudsOffset = Vector3.new(0, 3, 0)
            
            local lbl = Instance.new("TextLabel")
            lbl.Size = UDim2.new(1, 0, 1, 0)
            lbl.BackgroundTransparency = 1
            lbl.TextColor3 = Color3.fromRGB(255, 255, 255)
            lbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            lbl.TextStrokeTransparency = 0
            lbl.TextScaled = false
            lbl.TextSize = 10
            lbl.RichText = true
            
            lbl.Text = string.format(
                '<font color="rgb(255,0,0)">%s</font>\n<font color="rgb(0,255,0)">%s</font>\n<font color="rgb(255,255,0)">%s</font>',
                pet_name, info_txt, mut_txt or ""
            )
            
            lbl.Parent = bb
            cur_esp = bb
        end
        
        local function create_beam(target_part)
            local chr = lp.Character
            if not chr or not chr:FindFirstChild("HumanoidRootPart") or not target_part then return end
            
            if cur_beam then
                cur_beam:Destroy()
                if a0 then a0:Destroy() end
                if a1 then a1:Destroy() end
            end
            
            a0 = Instance.new("Attachment")
            a0.Parent = chr.HumanoidRootPart
            
            a1 = Instance.new("Attachment")
            a1.Parent = target_part
            
            local beam = Instance.new("Beam")
            beam.Attachment0 = a0
            beam.Attachment1 = a1
            beam.Width0 = 0.35
            beam.Width1 = 0.35
            beam.FaceCamera = true
            beam.LightInfluence = 0
            beam.Parent = line_folder
            
            cur_beam = beam
            
            task.spawn(function()
                while beam.Parent and esp_active do
                    local color = Color3.fromHSV((tick() % 5) / 5, 1, 1)
                    beam.Color = ColorSequence.new(color)
                    task.wait(0.05)
                end
            end)
        end
        
        local function find_best_pet()
            local plots_folder = ws:FindFirstChild("Plots")
            if not plots_folder then return end
            
            local best_val = -1
            local best_pet = nil
            
            for _, plot in ipairs(plots_folder:GetChildren()) do
                local podiums = plot:FindFirstChild("AnimalPodiums")
                if podiums then
                    for _, podium in ipairs(podiums:GetChildren()) do
                        local spawn_part = podium:FindFirstChild("Base") and podium.Base:FindFirstChild("Spawn")
                        if spawn_part then
                            local attach = spawn_part:FindFirstChild("Attachment")
                            local overhead = attach and attach:FindFirstChild("AnimalOverhead")
                            if overhead then
                                local name_lbl = overhead:FindFirstChild("DisplayName")
                                local gen_lbl = overhead:FindFirstChild("Generation")
                                local mut_lbl = overhead:FindFirstChild("Mutation")
                                if gen_lbl then
                                    local pet_name = name_lbl and name_lbl.Text or "Unknown"
                                    local gen_txt = gen_lbl.Text
                                    local mut_txt = mut_lbl and mut_lbl.Text or ""
                                    local mps = parse_mps(gen_txt)
                                    local gen_num = tonumber(gen_txt:match("%d+")) or 0
                                    local val = mps > 0 and mps or gen_num
                                    
                                    if val > best_val then
                                        best_val = val
                                        best_pet = {name = pet_name, info = gen_txt, mutation = mut_txt, part = spawn_part, value = val}
                                    end
                                end
                            end
                        end
                    end
                end
            end
            
            return best_pet
        end
        
        local function update_esp()
            local pet = find_best_pet()
            if pet then
                local need_update = false
                if not cur_esp then
                    need_update = true
                elseif cur_pet_info then
                    if pet.value > cur_pet_info.value or pet.mutation ~= cur_pet_info.mutation then
                        need_update = true
                    end
                end
                
                if need_update then
                    cur_pet_info = pet
                    create_esp(pet.name, pet.info, pet.mutation, pet.part)
                    create_beam(pet.part)
                end
            else
                if cur_esp then
                    cur_esp:Destroy()
                    cur_esp = nil
                    cur_pet_info = nil
                end
                if cur_beam then
                    cur_beam:Destroy()
                    cur_beam = nil
                end
            end
        end
        
        print('👁️ ESP iniciado...')
        while esp_active do
            update_esp()
            task.wait(0.3)
        end
        
        -- Cleanup
        if esp_folder then esp_folder:Destroy() end
        if line_folder then line_folder:Destroy() end
    end)
end

local function stop_esp()
    esp_active = false
    if esp_thread then
        task.cancel(esp_thread)
        esp_thread = nil
    end
    print('❌ ESP desativado')
end

local function criar_botao(num, pos_y)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 70, 0, 70)
    btn.Position = UDim2.new(0, 30, 0, pos_y)
    btn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    btn.Text = tostring(num)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 24
    btn.Font = Enum.Font.GothamBold
    btn.AutoButtonColor = false
    btn.Parent = btn_gui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = btn

    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(90, 90, 90)
    stroke.Thickness = 1.5
    stroke.Parent = btn

    local shadow = Instance.new("Frame")
    shadow.Size = UDim2.new(1, 6, 1, 6)
    shadow.Position = UDim2.new(0, -3, 0, -3)
    shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    shadow.BackgroundTransparency = 0.7
    shadow.ZIndex = btn.ZIndex - 1
    shadow.Parent = btn

    local shadow_corner = Instance.new("UICorner")
    shadow_corner.CornerRadius = UDim.new(0, 14)
    shadow_corner.Parent = shadow

    btn.MouseEnter:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
    end)

    btn.MouseLeave:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    end)

    btn.MouseButton1Down:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    end)

    btn.MouseButton1Up:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
        
        -- Funcionalidades dos botões
        if num == 1 then
            if farm_active then
                stop_farm()
                btn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
            else
                start_farm()
                btn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            end
        elseif num == 2 then
            if esp_active then
                stop_esp()
                btn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
            else
                start_esp()
                btn.BackgroundColor3 = Color3.fromRGB(0, 100, 255)
            end
        elseif num == 3 then
            print("Botão 3 - Funcionalidade futura")
        end
    end)
    
    return btn
end

criar_botao(1, 40)
criar_botao(2, 120)
criar_botao(3, 200)

-- Parte 3: Som de notificação
local snd_plr = game:GetService("Players")
local snd_local = snd_plr.LocalPlayer
local snd_gui = snd_local:WaitForChild("PlayerGui")

local audio = Instance.new("Sound")
audio.SoundId = table.concat({"rbxassetid://", "18886652611"})
audio.Volume = 5
audio.Looped = false
audio.Parent = snd_gui

audio:Play()
