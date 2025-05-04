------------------------------------------------------
-- Description ---------------------------------------
------------------------------------------------------
-- This script is designed to automate job switching for time-restricted missions and monitor credit .
-- I haven't tested it in languages other than Japanese.

-- requires the following addons:
-- Artisan 4.0.3.31
-- Ice's Cosmic Exploration 0.0.0.24

-------------------------------------------------------
---This section can be modified as needed -------------
-------------------------------------------------------

language = "jp" -- "jp", "en", "fr", "de"
weatherJob = "Armorer" -- Job to switch to during specific weather, or leave blank("")
margin = 2000 -- Credit limit margin
creditWarningInterval = 60 --real time, seconds
jobChangeWarningInterval = 30 -- real time, seconds
stackAvoidWarningInterval = 30 -- real time, seconds
Stop_if_Lunar_Credits_are_capped = false -- true or false
debugMessage = true -- true or false

-------------------------------------------------------
-- Job change schedule (in Eorzea time, minutes) ------
-------------------------------------------------------

jobChangeAheadMinutes = 20 --Eorzea time, minutes
JOB_SCHEDULE = {
    { startTime = 24 * 60 - jobChangeAheadMinutes, endTime = 2 * 60, job = "Carpenter" },
--    { startTime = 24 * 60 - jobChangeAheadMinutes, endTime = 2 * 60, job = "Alchemist" },
--    { startTime = 4 * 60 - jobChangeAheadMinutes, endTime = 6 * 60, job = "Culinarian" },
    { startTime = 4 * 60 - jobChangeAheadMinutes, endTime = 6 * 60, job = "Blacksmith" },
    { startTime = 8 * 60 - jobChangeAheadMinutes, endTime = 10 * 60, job = "Armorer" },
    { startTime = 12 * 60 - jobChangeAheadMinutes, endTime = 14 * 60, job = "Goldsmith" },
    { startTime = 16 * 60 - jobChangeAheadMinutes, endTime = 18 * 60, job = "Leatherworker" },
    { startTime = 20 * 60 - jobChangeAheadMinutes, endTime = 22 * 60, job = "Weaver" }
}

-------------------------------------------------------
-- Variables and Mappings -----------------------------
-------------------------------------------------------

lastCraftTime = os.clock()
lastCreditWarningTime = os.clock() - creditWarningInterval
lastJobChangeWarningTime = os.clock() - jobChangeWarningInterval

ID_TO_JOB = {
    [8] = "Carpenter",
    [9] = "Blacksmith",
    [10] = "Armorer",
    [11] = "Goldsmith",
    [12] = "Leatherworker",
    [13] = "Weaver",
    [14] = "Alchemist",
    [15] = "Culinarian"
}

GEARSET_MAP = {
    Carpenter = {
        jp = "木工師",
        en = "Carpenter",
        fr = "Menuisier",
        de = "Zimmerer"
    },
    Blacksmith = {
        jp = "鍛冶師",
        en = "Blacksmith",
        fr = "Forgeron",
        de = "Grobschmied"
    },
    Armorer = {
        jp = "甲冑師",
        en = "Armorer",
        fr = "Armurier",
        de = "Plattner"
    },
    Goldsmith = {
        jp = "彫金師",
        en = "Goldsmith",
        fr = "Orfèvre",
        de = "Goldschmied"
    },
    Leatherworker = {
        jp = "革細工師",
        en = "Leatherworker",
        fr = "Tanneur",
        de = "Gerber"
    },
    Weaver = {
        jp = "裁縫師",
        en = "Weaver",
        fr = "Couturier",
        de = "Weber"
    },
    Alchemist = {
        jp = "錬金術師",
        en = "Alchemist",
        fr = "Alchimiste",
        de = "Alchemist"
    },
    Culinarian = {
        jp = "調理師",
        en = "Culinarian",
        fr = "Cuisinier",
        de = "Gourmet"
    }
}

-------------------------------------------------------
-- Checking Functions
-------------------------------------------------------

function getEorzeaTime()
    return GetCurrentEorzeaHour(), GetCurrentEorzeaMinute(), GetCurrentEorzeaSecond()
end

-- 148 = moon dust, 49 = umbral wind
function isWeather()
    local weather = GetActiveWeatherID()
    return weather == 148 or weather == 49
end

function isOver()
    local moonCurrent, moonThreshold = getCreditValues(4)
    local cosmoCurrent, cosmoThreshold = getCreditValues(5)
    return (moonCurrent and moonCurrent >= moonThreshold) or (cosmoCurrent and cosmoCurrent >= cosmoThreshold)
end

function getCreditValues(rowIndex)
    if not IsAddonVisible("WKSHud") then return nil, nil end
    local rawCurrent = GetNodeText("WKSHud", rowIndex, 3)
    local rawLimit = GetNodeText("WKSHud", rowIndex, 1)
    local current = tonumber((rawCurrent:gsub(",", "")))
    local limit = tonumber((rawLimit:gsub(",", "")))
    if current and limit then
        return current, limit - margin
    end
    return nil, nil
end

function checkCredit()
    if isOver() and (os.clock() - lastCreditWarningTime) >= creditWarningInterval then
        yield("/echo Warning: Credits nearing max. <se.10>")
        lastCreditWarningTime = os.clock()
    end
end

-------------------------------------------------------
-- Job Change Logic
-------------------------------------------------------

function getScheduledJob(currentTime)
    for _, slot in ipairs(JOB_SCHEDULE) do
        if slot.startTime < slot.endTime then
            if currentTime >= slot.startTime and currentTime < slot.endTime then
                return slot.job
            end
        elseif currentTime >= slot.startTime or currentTime < slot.endTime then
            return slot.job
        end
    end
    return nil
end

function executeJobChange(jobName)
    local gearName = jobName
    local langKey = language:lower()
    local mapped = GEARSET_MAP[jobName]

    if mapped and mapped[langKey] then
        gearName = mapped[langKey]
    end

    if gearName then
        if debugMessage then
            yield("/echo Changing to gear set: " .. gearName)
        end
        yield("/gearset change " .. gearName)
    else
        if debugMessage then
            yield("/echo Job change failed: unknown job '" .. jobName .. "'. <se.10>")
        end
    end
end

function changeJob()
    local hour, minute, second = getEorzeaTime()
    local currentTime = hour * 60 + minute
    local scheduledJob = getScheduledJob(currentTime)

    local currentJobId = GetClassJobId()
    local currentJobName = ID_TO_JOB[currentJobId]

    local now = os.clock()
    -- Special weather job switching
    if isWeather() and weatherJob ~= "" then
        if currentJobName ~= weatherJob then
            if debugMessage then
                yield("/echo Special weather detected, switching to weather job: " .. weatherJob)
            end
            yield("/callback WKSMissionInfomation true 11")
            yield("/wait 0.5")
            executeJobChange(weatherJob)
        elseif (now - lastJobChangeWarningTime) >= jobChangeWarningInterval then
            if debugMessage then
                yield("/echo Weather job is already active during special weather: " .. weatherJob)
            end
            lastJobChangeWarningTime = os.clock()
        end
        return
    end

    -- No matching job for current time
    if not scheduledJob then
        if (now - lastJobChangeWarningTime) >= jobChangeWarningInterval then
            local timeStr = string.format("%02d:%02d.%02d", hour, minute, second)
            if debugMessage then
                yield("/echo No job found for ET " .. timeStr)
            end
            lastJobChangeWarningTime = os.clock()
        end
        return
    end

    -- Already on correct job
    if currentJobName == scheduledJob then
        if (now - lastJobChangeWarningTime) >= jobChangeWarningInterval then
            if debugMessage then
                yield("/echo Job is already correct: " .. currentJobName)
            end
            lastJobChangeWarningTime = os.clock()
        end
        return
    end

    -- execute job change (normal case)
    yield("/callback WKSMissionInfomation true 11")
    yield("/wait 0.5")
    executeJobChange(scheduledJob)
    yield("/wait 1")
    if IsAddonVisible("WKSMission") then
        yield("/callback WKSHud true 11")
        yield("/wait 0.5")
        yield("/callback WKSHud true 11")
    end
end

-------------------------------------------------------
-- Main Loop
-------------------------------------------------------

function main()
    while true do
        if IsCrafting() then
            lastCraftTime = os.clock()
        elseif (os.clock() - lastCraftTime) > stackAvoidWarningInterval then
            if debugMessage then
                yield("/echo No craft for more than "..stackAvoidWarningInterval.."seconds; restarting ICE. <se.1>")
            end

            if not IsAddonVisible("WKSMissionInfomation") then
                yield("/callback WKSHud true 11")
                yield("/wait 0.5")
                yield("/callback WKSMissionInfomation true 11")
            end

            yield("/ice stop")
            yield("/wait 0.5")
            yield("/ice start")
            lastCraftTime = os.clock()
        end

        -- Stop ICE and SND if Moon Credits are capped
        if Stop_if_Lunar_Credits_are_capped then
            local moonCurrent, moonLimit = getCreditValues(4)
            moonLimit = moonLimit + margin
            if moonCurrent and moonLimit and moonCurrent >= moonLimit then
                if debugMessage then
                    yield("/echo Moon Credits are capped! Stopping (ICE and )SND. <se.10>")
                end
--                yield("/ice stop")
                yield("/snd stop")
                return
            end
        end

        if not GetCharacterCondition(5) and not GetCharacterCondition(39) then
            changeJob()
        end

        checkCredit()

        yield("/wait 0.1")
    end
end

main()
