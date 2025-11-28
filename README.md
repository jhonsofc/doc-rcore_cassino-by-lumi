# rcore_casino - Documentação Completa

**Criado por Lumí Studios**

## 📋 Índice

1. [Visão Geral](#visao-geral)
2. [Instalação](#instalacao)
3. [Configuração de Framework](#configuracao-de-framework)
4. [Sistema de Pagamento](#sistema-de-pagamento)
5. [Configurações Disponíveis](#configuracoes-disponiveis)
6. [Jogos Disponíveis](#jogos-disponiveis)
7. [Personalização de Pagamentos](#personalizacao-de-pagamentos)
8. [Suporte a Frameworks](#suporte-a-frameworks)
9. [Sistema de Society](#sistema-de-society)
10. [Inventário](#inventario)
11. [Mapas Suportados](#mapas-suportados)
12. [Outras Configurações](#outras-configuracoes)
13. [Troubleshooting](#troubleshooting)
14. [Notas Finais](#notas-finais)

---

## 🎰 Visão Geral {#visao-geral}

O **rcore_casino** é um script completo de cassino para FiveM com múltiplos jogos, sistema de fichas, VIP, sociedade e muito mais. Este script suporta ESX, QBCore, Standalone e Custom frameworks.

### Características Principais

- ✅ 7 jogos diferentes (Slots, Roleta, Blackjack, Poker, Inside Track, Lucky Wheel, Bar)
- ✅ Sistema de fichas virtual ou por inventário
- ✅ Sistema VIP com benefícios exclusivos
- ✅ Sistema de Society (conta bancária do cassino)
- ✅ Suporte a múltiplos frameworks
- ✅ Suporte a múltiplos sistemas de inventário
- ✅ 13 tipos de mapas de cassino diferentes
- ✅ Sistema de horários de funcionamento
- ✅ Sistema de empregos (trabalhos no cassino)
- ✅ Sistema de logs e estatísticas

---

## 🚀 Instalação {#instalacao}

### Requisitos

- FiveM Server
- **Artefato versão 4752 ou mais recente** (obrigatório)
- MySQL (mysql-async ou ghmattimysql)
- Framework (ESX, QBCore ou Standalone)
- rcore_casino_assets (obrigatório)
- rcore_casino_interior (opcional, dependendo do mapa escolhido)

### Passos de Instalação

1. **Baixe e extraia o script** na pasta `resources`
2. **Configure o banco de dados** - Execute o SQL fornecido (se houver)
3. **Configure o `config.lua`** - Ajuste as configurações conforme necessário
4. **Configure o `config_server.lua`** - Configure notificações Discord (opcional)
5. **Adicione ao `server.cfg`**:
   ```
   ensure rcore_casino_assets
   ensure rcore_casino_interior  # Se usar mapa tipo 5
   ensure rcore_casino
   ```

---

## ⚙️ Configuração de Framework {#configuracao-de-framework}

### Framework Ativo

No arquivo `config.lua`, configure o framework:

```lua
Framework = {
    Active = 5, -- 1 = ESX, 2 = QBCore, 3 = Standalone, 4 = Custom, 5 = Auto Detect
    ES_EXTENDED_RESOURCE_NAME = "es_extended",
    ESX_SHARED_OBJECT = "esx:getSharedObject",
    QB_CORE_RESOURCE_NAME = "qb-core",
    QBCORE_SHARED_OBJECT = "QBCore:GetObject",
    STANDALONE_INITIAL_CHIPS = 100000
}
```

### Suporte a Creative Framework

**⚠️ IMPORTANTE:** O script **NÃO possui suporte nativo** para Creative Framework. Para usar com Creative, você precisa:

1. **Usar o modo Custom (Framework.Active = 4)**
2. **Editar `server/framework/custom.lua`** para implementar as funções do Creative
3. **Editar `client/framework/playerData.lua`** se necessário

#### Exemplo de Adaptação para Creative:

```lua
-- server/framework/custom.lua
if Framework.Active == 4 then
    ESX = {}
    
    ESX.GetPlayerFromId = function(source)
        local xPlayer = {}
        local player = exports['creative-core']:GetPlayer(source) -- Adaptar para Creative
        
        xPlayer.identifier = player.identifier
        xPlayer.getMoney = function()
            return player.getMoney() -- Adaptar função
        end
        -- ... implementar outras funções necessárias
        return xPlayer
    end
end
```

**Nota:** Você precisará adaptar todas as funções do ESX para usar as funções equivalentes do Creative Framework.

---

## 💰 Sistema de Pagamento {#sistema-de-pagamento}

### Como Funciona

O sistema de pagamento do cassino funciona da seguinte forma:

1. **Jogadores apostam fichas** nos jogos
2. **As fichas são removidas** do inventário/virtual do jogador
3. **Se habilitado, o dinheiro vai para a Society** (conta do cassino)
4. **Ao ganhar, as fichas são adicionadas** de volta ao jogador
5. **Se habilitado, o dinheiro sai da Society** para pagar o jogador

### Funções de Pagamento

As funções principais estão em `server/main/casino.lua`:

- `Pay(playerId, item, chips, game)` - Remove fichas do jogador
- `Win(playerId, item, chips, game)` - Adiciona fichas ao jogador
- `GetPlayerChips(playerId)` - Obtém fichas do jogador
- `GetPlayerMoney(playerId)` - Obtém dinheiro do jogador

### Personalização de Pagamentos

#### 1. Taxa de Câmbio

```lua
Config.ExchangeRate = 1 -- 1 ficha = 1$ (mínimo: 0.1)
```

#### 2. Usar Apenas Dinheiro

```lua
Config.UseOnlyMoney = false -- true = usa dinheiro direto, false = usa fichas
```

#### 3. Fichas Virtuais vs Inventário

```lua
Config.UseVirtualChips = true -- true = salva no DB, false = usa item de inventário
Config.ChipsInventoryItem = "casino_chips" -- Nome do item de ficha
```

#### 4. Dinheiro do Banco vs Dinheiro em Mão

```lua
Config.UseBankMoney = false -- true = usa banco, false = usa dinheiro em mão
```

#### 5. Limite de Saque Diário

```lua
Config.DailyWidthdrawLimit = 0 -- 0 = sem limite, ou valor máximo por dia
```

---

## 🎮 Configurações Disponíveis {#configuracoes-disponiveis}

### Configurações Gerais

```lua
Config.Locale = "en" -- Idioma (en, pt)
Config.MapType = 5 -- Tipo de mapa (1-13)
Config.UseTarget = false -- Usar sistema de target
Config.TargetZoneType = 1 -- 1=q_target, 2=bt_target, 3=qb-target, 4=ox_target
Config.RestrictControls = true -- Restringir controles dentro do cassino
Config.Debug = false -- Modo debug
```

### Configurações de Jogos

#### Inside Track (Corridas de Cavalos)

```lua
Config.IT_MAIN_EVENT_MIN_PLAYERS = 1 -- Mínimo de jogadores para evento principal
Config.IT_MAIN_EVENT_BETTING_TIME = 60 * 5 -- Tempo para apostas (5 minutos)
Config.IT_MAIN_EVENT_RACE_MAX_BET = 10000 -- Aposta máxima
Config.IT_MAIN_EVENT_RACE_MIN_BET = 10 -- Aposta mínima
Config.IT_MAIN_EVENT_RACE_WIN_CHANCE = 50 -- Chance de vitória (0-100)
Config.IT_LOCAL_RACE_WIN_CHANCE = 50 -- Chance de vitória corrida local
```

#### Roleta

```lua
Config.ROULETTE_JUNIOR_ENABLED = true -- Habilitar mesa Junior
Config.ROULETTE_JUNIOR_COORDS = {1004.790, 57.295, 68.432} -- Coordenadas
```

**Configurações em `const.lua`:**
- `RouletteTableDatas[0]` - Mesa Normal
- `RouletteTableDatas[2]` - Mesa Junior
- `RouletteTableDatas[3]` - Mesa High Stakes (VIP)

**Personalização:**
- `MinBetValue` / `MaxBetValue` - Limites de apostas
- `UnluckyRound` - A cada X rodadas, número menos favorável
- `TriggerUnluckyRoundFrom` - Ativa rodada azarada se custo > X

#### Blackjack

```lua
Config.BLACKJACK_JUNIOR_ENABLED = true
Config.BLACKJACK_JUNIOR_COORDS = {1004.183, 53.192, 68.432}
```

**Configurações em `const.lua`:**
- `BlackjackTableDatas[0]` - Mesa Normal
- `BlackjackTableDatas[2]` - Mesa Junior
- `BlackjackTableDatas[3]` - Mesa High Stakes (VIP)

**Personalização:**
- `MinimumBet` / `MaximumBet` - Limites de apostas
- `DealerBlackjackPossibility` - Chance do dealer fazer blackjack (%)
- `Dealer21Possibility` - Chance do dealer fazer 21 (%)

#### Poker

```lua
Config.POKER_JUNIOR_ENABLED = true
Config.POKER_JUNIOR_COORDS = {998.439, 61.031, 68.432}
```

**Configurações em `const.lua`:**
- `PokerTableDatas[0]` - Mesa Normal
- `PokerTableDatas[2]` - Mesa Junior
- `PokerTableDatas[3]` - Mesa High Stakes (VIP)

**Personalização:**
- `MinBetValueAntePlay` / `MaxBetValueAntePlay` - Limites Ante/Play
- `MinBetValuePairPlus` / `MaxBetValuePairPlus` - Limites Pair Plus
- `UnluckyRound` - A cada X rodadas, cartas ruins

**Bônus de Mãos (em `const.lua`):**
```lua
PokerHandBonus = {
    StraightFlush = 500,
    ThreeOfAKind = 400,
    Straight = 300,
    Flush = 200,
    Pair = 100
}
```

#### Slots

```lua
Config.SLOTS_1ST_PERSON = true -- Primeira pessoa ao jogar
```

**Personalização:** As configurações de slots estão no código do servidor (`server/games/slots.lua`). Você pode modificar:
- Probabilidades de vitória
- Multiplicadores de prêmios
- Tipos de máquinas

#### Lucky Wheel (Roda da Sorte)

```lua
Config.LUCKY_WHEEL_ENABLED = true
Config.LUCKY_WHEEL_COOLDOWN = (60 * 60 * 24) -- 24 horas
Config.LUCKY_WHEEL_PAY_TO_SPIN = 0 -- 0 = grátis, ou nome do item
Config.LUCKY_WHEEL_CAR_WINABLE = true -- Permite ganhar carro
Config.LUCKY_WHEEL_CAR_ONE_WINNER = true -- Apenas um ganhador por carro
```

**Prêmios em `const.lua`:**
- `LuckyWheelItems["Money1"]` até `["Money9"]` - Valores de dinheiro
- `LuckyWheelItems["Drinks"]` - Bebidas grátis
- `LuckyWheelItems["Vehicle"]` - Veículo do pódio
- `LuckyWheelItems["RP"]` - RP (se habilitado)

**Personalização:**
- `posibility` - Probabilidade de ganhar (1-100)
- `moneyReward` - Valor do prêmio em dinheiro
- `rotation` - Posição na roleta

### Cashier (Caixa)

```lua
Config.CASHIER_DAILY_BONUS = 1000 -- Bônus diário de visitante
Config.CASHIER_VIP_PRICE = 50000 -- Preço do VIP
Config.CASHIER_VIP_DURATION = (60 * 60 * 24) * 7 -- Duração VIP (7 dias)
Config.CASHIER_SHOW_SOCIETY_BALANCE = false -- Mostrar saldo da sociedade
```

### Bar

```lua
Config.BarShowSnacks = true -- Mostrar lanches no menu
```

**Itens do Bar em `const.lua`:**
- `CasinoInventoryItems` - Lista de bebidas e lanches
- `price` - Preço de cada item
- `consumable` - Tipo (1 = bebida, 2 = lanche)

---

## 🎲 Jogos Disponíveis {#jogos-disponiveis}

### 1. Slots (Caça-Níqueis)
- Múltiplas máquinas temáticas
- Apostas configuráveis
- Prêmios variados

### 2. Roulette (Roleta)
- Roleta americana (0 e 00)
- Apostas internas e externas
- Mesas Junior, Normal e High Stakes

### 3. Blackjack (21)
- Regras padrão de blackjack
- Split, Double Down
- Seven-Card Charlie
- Mesas Junior, Normal e High Stakes

### 4. Poker (Three Card Poker)
- Ante & Play
- Pair Plus (aposta lateral)
- Bônus por mãos especiais
- Mesas Junior, Normal e High Stakes

### 5. Inside Track (Corridas de Cavalos)
- Evento único (solo)
- Evento principal (multijogador)
- Sistema de odds
- Configuração de chance de vitória

### 6. Lucky Wheel (Roda da Sorte)
- Gire uma vez por dia (ou pague)
- Prêmios variados
- Veículo do pódio
- Cooldown configurável

### 7. Drinking Bar (Bar)
- Bebidas e lanches
- Sistema de embriaguez
- Bebidas grátis (VIP/Lucky Wheel)

---

## 💳 Personalização de Pagamentos {#personalizacao-de-pagamentos}

### Modificar Pagamentos dos Jogos

#### 1. Modificar Multiplicadores de Roleta

Edite `shared/utils.lua`, função `CalculateRoulettePlayerWinnings`:

```lua
-- Exemplo: Mudar Straight-up de 36x para 40x
if k <= 38 and winNumber == k then
    totalWinnings = totalWinnings + (v * 40) -- Era 36
end
```

#### 2. Modificar Pagamentos de Blackjack

Edite `server/games/blackjack.lua` para alterar:
- Pagamento de Blackjack (padrão: 3:2)
- Pagamentos normais (padrão: 1:1)

#### 3. Modificar Pagamentos de Poker

Edite `const.lua`, `PokerHandBonus`:

```lua
PokerHandBonus = {
    StraightFlush = 500, -- Aumentar/diminuir valores
    ThreeOfAKind = 400,
    -- ...
}
```

#### 4. Modificar Pagamentos de Inside Track

As odds determinam os pagamentos. Cavalos com odds menores pagam menos, mas têm mais chance de ganhar.

#### 5. Modificar Pagamentos de Slots

Edite `server/games/slots.lua` para modificar:
- Probabilidades de cada símbolo
- Multiplicadores de prêmios
- Combinações vencedoras

### Sistema de Society (Conta do Cassino)

Quando habilitado, todos os pagamentos passam pela conta da sociedade:

```lua
Config.EnableSociety = true
Config.SocietyName = "society_casino"
Config.SocietyLimitFromBalance = 10000 -- Limite mínimo
Config.SocietyLimitPayoutPercentage = 35 -- % reduzido se abaixo do limite
```

**Como funciona:**
- Jogadores apostam → Dinheiro vai para a Society
- Jogadores ganham → Dinheiro sai da Society
- Se Society < `SocietyLimitFromBalance` → Pagamentos reduzidos em `SocietyLimitPayoutPercentage`%

---

## 🔧 Suporte a Frameworks {#suporte-a-frameworks}

### Frameworks Suportados

1. **ESX** (Framework.Active = 1)
2. **QBCore** (Framework.Active = 2)
3. **Standalone** (Framework.Active = 3)
4. **Custom** (Framework.Active = 4)
5. **Auto Detect** (Framework.Active = 5)

### Creative Framework

**⚠️ NÃO TEM SUPORTE NATIVO**

Para usar com Creative:
1. Configure `Framework.Active = 4` (Custom)
2. Edite `server/framework/custom.lua`
3. Implemente as funções do Creative seguindo o padrão do ESX

---

## 🏦 Sistema de Society {#sistema-de-society}

### Sistemas Suportados

```lua
Config.SocietyFramework = Society.AutoChoose -- Auto detecta
-- Ou escolha manualmente:
-- Society.QbBanking
-- Society.QbBossMenu
-- Society.QbManagement
-- Society.RenewedBanking
-- Society.OkokBanking
-- Society.Management710
-- Society.FdBanking
-- Society.EsxAddonAccount
-- Society.SnipeBanking
-- Society.ManagementFunds
-- Society.Tgg
-- Society.WasabiBanking
-- Society.Origen
-- Society.Tgiann
-- Society.QsBanking
```

### Configuração

```lua
Config.EnableSociety = false -- Habilitar sistema
Config.SocietyAutoInstall = true -- Instalar automaticamente
Config.SocietyName = "society_casino" -- Nome da conta
```

---

## 📦 Inventário {#inventario}

### Sistemas Suportados

```lua
Config.InventoryResource = Inventory.AutoChoose -- Auto detecta
-- Ou escolha manualmente:
-- Inventory.OX (ox_inventory)
-- Inventory.LJ (lj_inventory)
-- Inventory.MF (mf_inventory)
-- Inventory.PS (ps_inventory)
-- Inventory.QS (qs-inventory)
-- Inventory.CodeM (codem-inventory)
-- Inventory.Framework (inventário do framework)
-- Inventory.Origen
-- Inventory.Tgiann
-- Inventory.jpr
```

### Configuração de Fichas

```lua
Config.UseVirtualChips = true -- true = salva no DB, false = usa item
Config.ChipsInventoryItem = "casino_chips" -- Nome do item
Config.MoneyInventoryItemName = nil -- Item de dinheiro (nil = usa framework)
```

---

## 🗺️ Mapas Suportados {#mapas-suportados}

```lua
Config.MapType = 5 -- Escolha o tipo:
-- 1: UncleJust Casino / DlcIplLoader
-- 2: Gabz Casino
-- 3: NoPixel Casino
-- 4: k4mb1 casino
-- 5: GTA:O Interior (rcore_casino_interior) ⭐ RECOMENDADO
-- 6: Underground Casino PALETO
-- 7: K4MB1 old enterable casino
-- 8: patoche_casino
-- 9: Clawles: Casino Los Santos
-- 10: Molo Casino
-- 11: Jurassic Jackpot Casino
-- 12: Energy Atlantis Casino
-- 13: HG Designs: The Town Resort & Casino
```

### Notas Importantes

- Se usar mapa tipo 5, mantenha `rcore_casino_interior` habilitado
- Se usar outros mapas, desabilite `rcore_casino_interior` e remova das dependências
- Alguns mapas requerem arquivos extras (veja comentários no `config.lua`)

---

## 🎯 Outras Configurações {#outras-configuracoes}

### Horários de Funcionamento

```lua
Config.OpeningHours = {
    [1] = {-1}, -- Domingo: Aberto 24h
    [2] = {0, 1, 2, 3, 4, 5, 6}, -- Segunda: 0h às 6h
    -- ... outros dias
}
-- {-1} = Aberto 24h
-- {0, 1, 2} = Aberto das 0h às 2h
```

### Sistema de Notificações

```lua
Config.NotifySystem = 1
-- 1: GTAV padrão
-- 2: okokNotify
-- 3: esx_notify
-- 4: qb_notify
-- 5: ox_notify
-- 6: origen help notify
```

### Sistema de Embriaguez

```lua
Config.DrunkSystem = 1
-- 1: Auto-detect / built-in
-- 2: esx_status
-- 3: evidence:client:SetStatus
-- 4: rcore_drunk
```

### Sistema de Trabalho

```lua
Config.JobName = "casino" -- Nome do trabalho
Config.BossGrade = 2 -- Nível máximo (chefe)
Config.BossName = "boss" -- Nome do cargo de chefe
```

### Teleporte

```lua
Config.LeaveThroughTeleport = false -- Sair por teleporte
Config.EnterPosition = vector3(2469.584473, -280.015869, -58.267620)
Config.LeavePosition = vector3(919.127380, 51.120274, 80.898659)
Config.LeaveArea = vector3(2468.992188, -287.276459, -58.267506)
```

### UI

```lua
Config.UIFontName = nil -- Fonte customizada
Config.ShowHowToPlayUI = 1 -- 0=desabilitado, 1=uma vez, 2=sempre
Config.ShowChipsHud = true -- HUD de fichas
Config.UseNUIHUD = false -- HUD NUI customizado
```

### Peds e Ambiente

```lua
Config.CASINO_ENABLE_AMBIENT_PEDS = true -- Peds parados
Config.CASINO_ENABLE_AMBIENT_PEDS_SLOTS = false -- Peds jogando slots
Config.CASINO_AMBIENT_PEDS_DENSITY = 3 -- 1=poucos, 2=médio, 3=todos
```

### Habilitar/Desabilitar Jogos

```lua
GameStates = {{
    activity = "slots",
    title = "Slot Machines",
    enabled = true -- false para desabilitar
}, {
    activity = "roulette",
    title = "Roulette",
    enabled = true
}, -- ... outros jogos
}
```

---

## 🐛 Troubleshooting {#troubleshooting}

### Problemas Comuns

1. **Script não carrega**
   - Verifique se `rcore_casino_assets` está iniciado
   - Verifique dependências no `fxmanifest.lua`

2. **Fichas não aparecem**
   - Verifique `Config.UseVirtualChips`
   - Verifique se o item `casino_chips` existe no inventário

3. **Pagamentos não funcionam**
   - Verifique se a Society está configurada corretamente
   - Verifique se há dinheiro suficiente na Society

4. **Framework não detectado**
   - Configure `Framework.Active` manualmente
   - Verifique nomes dos recursos no `config.lua`

5. **Mapa não aparece**
   - Verifique `Config.MapType`
   - Verifique se o mapa está instalado
   - Verifique se `rcore_casino_interior` está habilitado (se necessário)

---

## 📝 Notas Finais {#notas-finais}

- **Versão do Script:** 1.8.1
- **Game Build Mínimo:** 2060
- **Artefato Mínimo:** Versão 4752 ou mais recente (obrigatório)
- **Requer OneSync:** Sim
- **Requer MySQL:** Sim (mysql-async ou ghmattimysql)

### Arquivos Editáveis (Não Protegidos)

- `config.lua`
- `config_server.lua`
- `coords.lua`
- `const.lua` (parcialmente)
- `locales/*.lua`
- `server/main/casino.lua` (parcialmente)
- `server/main/society.lua`
- `server/utils/*.lua`

### Suporte

Para suporte e documentação oficial, visite:
- Documentação: https://documentation.rcore.cz/paid-resources/rcore_casino

---

**Desenvolvido por Lumí Studios**

