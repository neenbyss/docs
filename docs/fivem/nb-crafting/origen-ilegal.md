# Origen Ilegal (bandas)

Con **Origen Ilegal**, las restricciones de estaciones y recetas pueden usar la **banda** del jugador (gang ID = job, nivel = grade). Solo cambian **6 funciones** en `bridge/framework.lua`.

**Opciones:**

1. **Reemplazar el archivo:** En `fxmanifest.lua` cambia `'bridge/framework.lua'` por `'bridge/framework_origen_ilegal.lua'` en `shared_scripts`.
2. **Solo pegar el código:** En `bridge/framework.lua` sustituye las 3 funciones **servidor** (GetPlayerJob, GetPlayerGrade, GetPlayerJobData) por el bloque SERVER de abajo, y las 3 **cliente** por el bloque CLIENT.

---

## Bloque SERVER (dentro de `if IsDuplicityVersion()`)

```lua
    function Bridge.GetPlayerJob(source)
        local gangID = exports["origen_ilegal"]:GetGangID(source)
        if gangID and gangID ~= false then
            return tostring(gangID)
        end
        local player = Bridge.GetPlayer(source)
        if not player then return nil end
        if Bridge.Framework == 'ESX' then
            return player.job.name
        elseif Bridge.Framework == 'QBCore' then
            return player.PlayerData.job.name
        end
        return nil
    end

    function Bridge.GetPlayerGrade(source)
        local gangID = exports["origen_ilegal"]:GetGangID(source)
        if gangID and gangID ~= false then
            local gangData = exports["origen_ilegal"]:GetGangData(gangID)
            if gangData and gangData.gangLevel then
                return gangData.gangLevel or 0
            end
        end
        local player = Bridge.GetPlayer(source)
        if not player then return 0 end
        if Bridge.Framework == 'ESX' then
            return player.job.grade
        elseif Bridge.Framework == 'QBCore' then
            return player.PlayerData.job.grade.level
        end
        return 0
    end

    function Bridge.GetPlayerJobData(source)
        local gangID = exports["origen_ilegal"]:GetGangID(source)
        if gangID and gangID ~= false then
            local gangData = exports["origen_ilegal"]:GetGangData(gangID)
            if gangData then
                return {
                    job = tostring(gangID),
                    grade = gangData.gangLevel or 0
                }
            end
        end
        local player = Bridge.GetPlayer(source)
        if not player then return {job = nil, grade = 0} end
        if Bridge.Framework == 'ESX' then
            return { job = player.job.name, grade = player.job.grade }
        elseif Bridge.Framework == 'QBCore' then
            return { job = player.PlayerData.job.name, grade = player.PlayerData.job.grade.level }
        end
        return {job = nil, grade = 0}
    end
```

---

## Bloque CLIENT (dentro del `else`)

```lua
    function Bridge.GetPlayerJob()
        local gangID = exports["origen_ilegal"]:GetGangID()
        if gangID and gangID ~= false then
            return tostring(gangID)
        end
        local playerData = Bridge.GetPlayerData()
        if not playerData or not playerData.job then return nil end
        return playerData.job.name
    end

    function Bridge.GetPlayerGrade()
        local gangID = exports["origen_ilegal"]:GetGangID()
        if gangID and gangID ~= false then
            return 0
        end
        local playerData = Bridge.GetPlayerData()
        if not playerData or not playerData.job then return 0 end
        if Bridge.Framework == 'ESX' then
            return playerData.job.grade or 0
        elseif Bridge.Framework == 'QBCore' then
            return playerData.job.grade.level or 0
        end
        return 0
    end

    function Bridge.GetPlayerJobData()
        local gangID = exports["origen_ilegal"]:GetGangID()
        if gangID and gangID ~= false then
            return { job = tostring(gangID), grade = 0 }
        end
        local playerData = Bridge.GetPlayerData()
        if not playerData or not playerData.job then return {job = nil, grade = 0} end
        if Bridge.Framework == 'ESX' then
            return { job = playerData.job.name, grade = playerData.job.grade or 0 }
        elseif Bridge.Framework == 'QBCore' then
            return { job = playerData.job.name, grade = playerData.job.grade.level or 0 }
        end
        return {job = nil, grade = 0}
    end
```

En el panel admin usa el **ID de banda** como “job” y el **nivel de banda** como “grade”. Ten **origen_ilegal** iniciado antes que nb-crafting.
