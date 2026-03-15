# ForceSense — Lua Scripting API Reference

> **English** | **Русский** — bilingual documentation  
> All scripts are placed in `C:\force\scripts\`. Extension: `.lua`

---

# Table of Contents / Содержание

1. [Script Lifecycle](#script-lifecycle--жизненный-цикл-скрипта)
2. [render — Drawing / Рисование](#render--drawing--рисование)
3. [image — Textures / Текстуры](#image--textures--текстуры)
4. [sound — Audio / Аудио](#sound--audio--аудио)
5. [http — HTTP Requests / HTTP-запросы](#http--http-requests--http-запросы)
6. [ffi — Foreign Function Interface](#ffi--foreign-function-interface)
7. [game — Game Data / Данные игры](#game--game-data--данные-игры)
8. [ui — UI Builder / Построитель UI](#ui--ui-builder--построитель-ui)
9. [imgui — ImGui Widgets](#imgui--imgui-widgets)
10. [memory — Memory Read/Write / Чтение и запись памяти](#memory--memory-readwrite--чтение-и-запись-памяти)
11. [settings — Settings Access / Доступ к настройкам](#settings--settings-access--доступ-к-настройкам)
12. [input — Input / Ввод](#input--input--ввод)
13. [utils — Utilities / Утилиты](#utils--utilities--утилиты)
14. [client — Low-level CS2 Pointers](#client--low-level-cs2-pointers)
15. [log — Console Logging / Логирование](#log--console-logging--логирование)

---

## Script Lifecycle / Жизненный цикл скрипта

**EN:** Every script can define up to three global callback functions. The engine calls them automatically.

**RU:** Каждый скрипт может определить до трёх глобальных callback-функций, которые движок вызывает автоматически.

```lua
function OnLoad()
    -- Called once when the script is loaded.
    -- Вызывается один раз при загрузке скрипта.
end

function OnFrame()
    -- Called every rendered frame. Use render.* and image.* here.
    -- Вызывается каждый кадр. Используй render.* и image.* здесь.
end

function OnUnload()
    -- Called when the script is unloaded.
    -- Вызывается при выгрузке скрипта.
end
```

**Global variables / Глобальные переменные:**

| Variable | Type | Description |
|---|---|---|
| `__script_path` | string | Full path of the script file / Полный путь к файлу скрипта |
| `__script_name` | string | File name only / Только имя файла |
| `NULL` | integer | 0 — null pointer constant / константа нулевого указателя |

---

## render — Drawing / Рисование

**EN:** All `render.*` functions must be called inside `OnFrame()`. Colors are Lua tables `{r, g, b, a}` with values in the range `0.0–1.0`. Use `render.color()` to build one conveniently.

**RU:** Все функции `render.*` должны вызываться внутри `OnFrame()`. Цвета — Lua-таблицы `{r, g, b, a}` со значениями `0.0–1.0`. Для удобства используй `render.color()`.

```lua
local red   = render.color(1, 0, 0, 1)
local white = render.color(1, 1, 1, 0.8)
```

### render.color(r, g, b [, a]) → table

Creates a color table. Alpha defaults to `1.0` if omitted.  
Создаёт таблицу цвета. Альфа по умолчанию `1.0`.

---

### render.line(x1, y1, x2, y2, color [, thickness])

Draws a line from `(x1,y1)` to `(x2,y2)`.  
Рисует линию.

```lua
render.line(100, 100, 400, 200, render.color(1,1,0,1), 2)
```

---

### render.rect(x, y, w, h, color [, thickness [, rounding]])

Draws a hollow rectangle outline.  
Рисует контур прямоугольника.

---

### render.rect_filled(x, y, w, h, color [, rounding])

Draws a filled rectangle.  
Рисует закрашенный прямоугольник.

```lua
render.rect_filled(50, 50, 200, 100, render.color(0.1, 0.1, 0.1, 0.9), 4)
```

---

### render.circle(x, y, radius, color [, segments [, thickness]])

Draws a hollow circle. Default segments = 32.  
Рисует окружность. Сегментов по умолчанию 32.

---

### render.circle_filled(x, y, radius, color [, segments])

Draws a filled circle.  
Рисует закрашенный круг.

---

### render.text(x, y, text, color [, size])

Draws text at screen position. `size` defaults to the current ImGui font size.  
Рисует текст. `size` по умолчанию — текущий размер шрифта ImGui.

```lua
render.text(10, 10, "Hello World", render.color(1,1,1,1), 14)
```

---

### render.triangle(x1,y1, x2,y2, x3,y3, color [, thickness])
### render.triangle_filled(x1,y1, x2,y2, x3,y3, color)

Draws a triangle outline or filled triangle.  
Рисует контур или закрашенный треугольник.

---

### render.get_screen_size() → w, h

Returns screen width and height in pixels.  
Возвращает ширину и высоту экрана в пикселях.

---

### render.world_to_screen(wx, wy, wz) → sx, sy, visible

Projects a 3D world position to 2D screen coordinates.  
Returns `sx, sy, visible` — visibility is `true` if the point is on screen.  
Проецирует мировые координаты на экран. Возвращает `sx, sy, visible`.

```lua
local sx, sy, vis = render.world_to_screen(x, y, z)
if vis then
    render.circle_filled(sx, sy, 5, render.color(1,0,0,1))
end
```

---

## image — Textures / Текстуры

**EN:** Supports PNG, JPG, BMP, TGA, GIF (first frame). Textures are cached — calling `image.load()` with the same path twice returns the same handle.

**RU:** Поддерживает PNG, JPG, BMP, TGA, GIF (первый кадр). Текстуры кешируются — повторный вызов `image.load()` с тем же путём возвращает уже готовый handle.

---

### image.load(path) → handle | nil, errmsg

Loads an image from a file path. Returns an integer handle on success, or `nil` + error message on failure.  
Загружает изображение из файла.

```lua
local img, err = image.load("C:\\force\\img\\logo.png")
if not img then log.print("Error: " .. err) end
```

---

### image.load_memory(data) → handle | nil, errmsg

Loads an image from raw bytes (a Lua string). Useful with `http` responses.  
Загружает изображение из сырых байт Lua-строки.

---

### image.draw(handle, x, y [, w, h [, alpha]])

Draws the image at `(x, y)`. If `w`/`h` are omitted, uses the original texture size. `alpha` is `0.0–1.0`.  
Рисует изображение. Если `w`/`h` не указаны — используется оригинальный размер.

```lua
image.draw(img, 100, 100)              -- original size
image.draw(img, 100, 100, 64, 64)      -- scaled
image.draw(img, 100, 100, 64, 64, 0.8) -- scaled + 80% opacity
```

---

### image.draw_rounded(handle, x, y, w, h, rounding [, alpha])

Draws the image with rounded corners.  
Рисует изображение со скруглёнными углами.

---

### image.draw_quad(handle, x1,y1, x2,y2, x3,y3, x4,y4 [, alpha])

Draws the image mapped to an arbitrary quadrilateral (perspective warp).  
Рисует изображение в произвольный четырёхугольник (перспективное растяжение).

---

### image.size(handle) → w, h

Returns the texture dimensions in pixels.  
Возвращает размеры текстуры в пикселях.

---

### image.valid(handle) → bool

Returns `true` if the handle points to a loaded texture.  
Возвращает `true`, если handle указывает на загруженную текстуру.

---

### image.free(handle)

Releases a single texture and removes it from the cache.  
Освобождает одну текстуру и удаляет её из кеша.

---

### image.free_all()

Releases all loaded textures. Call this in `OnUnload()` if you loaded images.  
Освобождает все загруженные текстуры.

---

## sound — Audio / Аудио

**EN:** Supports WAV, MP3, WMA, MIDI, AIFF — anything Windows MCI handles. Up to **16 channels** can play simultaneously.

**RU:** Поддерживает WAV, MP3, WMA, MIDI, AIFF — всё, что умеет MCI Windows. До **16 каналов** одновременно.

---

### sound.play(path [, loop]) → channel_id

Fire-and-forget playback. Returns channel id (or `-1` if all channels are busy).  
Воспроизведение «выстрелил-забыл». Возвращает id канала.

```lua
sound.play("C:\\force\\snd\\alert.wav")
sound.play("C:\\force\\snd\\bg.mp3", true) -- looped / зациклить
```

---

### sound.play_sync(path)

Blocking (synchronous) playback. Stops the calling thread until playback finishes. WAV only.  
Синхронное воспроизведение. Блокирует поток. Только WAV.

---

### sound.open(path) → channel_id | nil

Opens a file on a channel without starting playback. Returns `nil` if no free channels.  
Открывает файл на канале без воспроизведения.

---

### sound.start(ch [, loop])

Starts playback on channel `ch` from position 0.  
Начинает воспроизведение с начала.

---

### sound.pause(ch) / sound.resume(ch) / sound.stop(ch)

Pauses, resumes, or stops a channel.  
Пауза / возобновление / остановка канала.

---

### sound.seek(ch, ms)

Seeks to position `ms` (milliseconds).  
Перематывает на позицию `ms` миллисекунд.

---

### sound.pos(ch) → ms

Returns the current playback position in milliseconds.  
Возвращает текущую позицию в миллисекундах.

---

### sound.length(ch) → ms

Returns the total track duration in milliseconds.  
Возвращает полную длительность трека.

---

### sound.status(ch) → string

Returns `"playing"`, `"paused"`, `"stopped"`, `"closed"`, or `"error"`.  
Возвращает строку статуса канала.

---

### sound.is_playing(ch) → bool

Returns `true` if the channel is currently playing.  
Возвращает `true`, если канал воспроизводит звук.

---

### sound.volume(ch, 0..1000)

Sets the volume of a channel. Range: `0` (silent) to `1000` (maximum). Note: not all MCI types support this — WAV definitely does.  
Устанавливает громкость. Диапазон: `0–1000`. Не все типы MCI поддерживают.

---

### sound.close(ch) / sound.close_all() / sound.stop_all()

Closes one channel / all channels / stops all without closing.  
Закрывает один / все каналы / останавливает все без закрытия.

---

### sound.beep(freq_hz, duration_ms)

Emits a system beep tone.  
Системный звуковой сигнал.

```lua
sound.beep(880, 200) -- 880 Hz for 200 ms
```

---

### sound.system(name)

Plays a Windows system sound by alias name.  
Воспроизводит системный звук Windows по имени.

Valid names / Допустимые имена: `"SystemAsterisk"`, `"SystemExclamation"`, `"SystemHand"`, `"SystemQuestion"`, `"SystemDefault"`, `"SystemExit"`, `".Default"`

```lua
sound.system("SystemAsterisk")
```

---

## http — HTTP Requests / HTTP-запросы

**EN:** Uses Windows WinHTTP. Requests run on the caller's thread — use Lua coroutines for non-blocking behavior.

**RU:** Использует WinHTTP. Запросы выполняются в потоке вызывающего — используй корутины для асинхронности.

---

### http.get(url) → ok, body

Performs a GET request. Returns `ok` (bool) and `body` (string) or error message.  
Выполняет GET-запрос.

```lua
local ok, body = http.get("https://api.example.com/data")
if ok then log.print(body) end
```

---

### http.post(url, body [, content_type]) → ok, body

Performs a POST request. `content_type` defaults to `"application/x-www-form-urlencoded"`.  
Выполняет POST-запрос.

```lua
local ok, resp = http.post("https://api.example.com/submit",
    '{"key":"val"}', "application/json")
```

---

### http.request(table) → ok, body, status_code

Full-featured request. Table fields:

| Field | Type | Description |
|---|---|---|
| `url` | string | Required / обязательно |
| `method` | string | Default `"GET"` |
| `body` | string | Request body / тело запроса |
| `headers` | table | `{["Name"]="Value"}` |
| `timeout` | number | Milliseconds, default 10000 |

Returns three values: `ok`, `body/error`, `status_code`.  
Возвращает три значения: `ok`, `body/ошибка`, `status_code`.

```lua
local ok, body, code = http.request({
    url     = "https://api.example.com/v1/users",
    method  = "GET",
    headers = { ["Authorization"] = "Bearer TOKEN" },
    timeout = 5000
})
```

---

## ffi — Foreign Function Interface

**EN:** Allows loading Windows DLLs, resolving symbols, calling functions, and reading/writing arbitrary memory. All addresses are passed as Lua integers (64-bit on x64).

**RU:** Позволяет загружать DLL Windows, находить адреса функций, вызывать их и читать/записывать произвольную память. Адреса передаются как Lua-числа (64-bit на x64).

---

### Library Loading / Загрузка библиотек

```lua
-- Load a DLL (GC will call FreeLibrary automatically)
-- Загрузить DLL (GC вызовет FreeLibrary автоматически)
local lib = ffi.load("user32.dll")

-- Get the address of an exported function
-- Получить адрес экспортируемой функции
local addr = ffi.sym(lib, "MessageBoxA")

-- Explicit free (optional)
-- Явное освобождение (опционально)
ffi.free(lib)
```

---

### ffi.get_proc(dll, func) → address | nil

Shorthand: loads the DLL, resolves the symbol, frees the DLL, returns the address.  
Сокращение: грузит DLL, находит символ, освобождает DLL, возвращает адрес.

---

### ffi.module(dll) → base_address | nil

Returns the base address of an already-loaded module (`GetModuleHandleA`). Does not load the DLL.  
Возвращает базовый адрес уже загруженного модуля без загрузки.

---

### Calling Functions / Вызов функций

All call variants accept a table of up to 8 arguments (integers, numbers, or strings — strings are passed as their pointer).

Все варианты вызова принимают таблицу до 8 аргументов.

```lua
-- Returns an integer
local ret = ffi.call_int(addr, {arg1, arg2, arg3})

-- Returns a double
local ret = ffi.call_float(addr, {arg1, arg2})

-- Reads the returned char* as a Lua string
local str = ffi.call_str(addr, {arg1})
```

---

### Memory Read / Чтение памяти

```lua
local v  = ffi.read_u8 (addr)          -- uint8
local v  = ffi.read_u32(addr)          -- uint32
local v  = ffi.read_u64(addr)          -- uint64
local v  = ffi.read_f32(addr)          -- float
local v  = ffi.read_f64(addr)          -- double
local s  = ffi.read_str(addr, len)     -- raw bytes as string / сырые байты
local s  = ffi.to_string(addr)         -- null-terminated C string / C-строка
local p  = ffi.deref(addr)             -- read pointer (8 bytes) / прочитать указатель
```

---

### Memory Write / Запись памяти

```lua
ffi.write_u8 (addr, val)
ffi.write_u32(addr, val)
ffi.write_u64(addr, val)
ffi.write_f32(addr, val)
ffi.write_f64(addr, val)
ffi.write_str(addr, str)  -- write raw bytes from Lua string / записать байты Lua-строки
```

---

## game — Game Data / Данные игры

**EN:** High-level CS2 data access. All player indices are **1-based**. Functions return `0` / `nil` / `false` when data is unavailable.

**RU:** Высокоуровневый доступ к данным CS2. Индексы игроков — **начиная с 1**. При недоступности данных функции возвращают `0` / `nil` / `false`.

---

### Local Player / Локальный игрок

```lua
game.get_local_player_hp()              -- → int (health)
game.get_local_player_armor()           -- → int
game.get_local_player_team()            -- → int (2=T, 3=CT)
game.get_local_player_name()            -- → string
game.get_local_player_pos()             -- → x, y, z (world position)
game.get_local_player_velocity()        -- → vx, vy, vz
game.get_local_player_speed()           -- → float (units/s)
game.get_local_player_flags()           -- → int (bit flags)
game.get_local_player_eye_pos()         -- → x, y, z
game.get_local_player_view_angles()     -- → pitch, yaw
game.get_local_player_shots_fired()     -- → int
game.get_local_player_bone(bone_id)     -- → x, y, z (bone world position)
game.get_local_player_flash()           -- → float (flash amount 0..255)

game.is_local_player_alive()            -- → bool
game.is_local_player_scoped()           -- → bool
game.is_local_player_flashed()          -- → bool
game.is_local_player_grounded()         -- → bool
game.is_local_player_defusing()         -- → bool
game.is_local_player_ducking()          -- → bool
game.has_helmet()                       -- → bool
game.has_defuser()                      -- → bool
```

---

### Other Players / Другие игроки

```lua
game.get_player_count()                 -- → int (total slots scanned)
game.get_player_hp(idx)                 -- → int
game.get_player_armor(idx)              -- → int
game.get_player_team(idx)               -- → int
game.get_player_name(idx)               -- → string
game.get_player_pos(idx)                -- → x, y, z
game.get_player_bone(idx, bone_id)      -- → x, y, z
game.get_player_velocity(idx)           -- → vx, vy, vz
game.get_player_speed(idx)              -- → float
game.get_player_dist(idx)               -- → float (distance to local player)
game.get_player_flash_amount(idx)       -- → float
game.get_player_head_screen(idx)        -- → sx, sy, visible
game.get_player_screen(idx)             -- → sx, sy, visible  (feet)
game.get_player_bone_screen(idx, bone)  -- → sx, sy, visible
game.get_player_eye_pos(idx)            -- → x, y, z
game.get_player_kills(idx)              -- → int
game.get_player_deaths(idx)             -- → int
game.get_player_assists(idx)            -- → int
game.get_player_ping(idx)               -- → int
game.get_player_mvps(idx)               -- → int
game.get_player_score(idx)              -- → int
game.get_player_money(idx)              -- → int
game.get_player_shots_fired(idx)        -- → int

game.is_player_alive(idx)               -- → bool
game.is_player_scoped(idx)              -- → bool
game.is_player_enemy(idx)               -- → bool
game.is_player_flashed(idx)             -- → bool
game.is_player_grounded(idx)            -- → bool
game.is_player_bot(idx)                 -- → bool
game.is_player_ducking(idx)             -- → bool
game.is_player_in_air(idx)              -- → bool
game.player_has_helmet(idx)             -- → bool

-- Returns a table of all enemy player indices
-- Возвращает таблицу индексов всех врагов
game.get_all_enemies()                  -- → {idx, ...}

-- Returns a table of all valid player indices
-- Возвращает таблицу всех действующих игроков
game.get_all_players()                  -- → {idx, ...}

game.get_alive_count()                  -- → int
```

---

### Aim / Прицеливание

```lua
-- Move aim toward screen point (sx, sy) with given speed
-- Переместить прицел к экранной точке с заданной скоростью
game.aim_at(sx, sy, speed)

-- Aim at specific bone of player idx
-- Прицелиться в кость игрока
game.aim_at_bone(idx, bone_id, speed)

-- Closest enemy (screen-distance from center)
-- Ближайший враг (расстояние от центра экрана)
game.get_closest_enemy()                -- → idx, dist

-- Closest enemy within FOV degrees (screen FOV)
game.get_closest_enemy_fov(fov)         -- → idx, fov_dist

-- Closest enemy by 3D world distance
game.get_closest_enemy_3d()             -- → idx, dist

-- Index of player currently under the crosshair
game.get_crosshair_target()             -- → idx or 0
```

---

### Game State / Состояние игры

```lua
game.get_fps()                          -- → float
game.get_screen_size()                  -- → w, h
game.get_screen_center()                -- → cx, cy
game.is_game_focused()                  -- → bool
game.world_to_screen(wx, wy, wz)        -- → sx, sy, visible
game.get_view_matrix()                  -- → table[16] (column-major)
game.get_time()                         -- → float (seconds since epoch)
game.get_tick()                         -- → int (GetTickCount)

-- Round / bomb
game.is_bomb_planted()                  -- → bool
game.get_bomb_time()                    -- → float (seconds remaining)
game.get_round_number()                 -- → int
game.is_warmup()                        -- → bool
game.is_freeze_time()                   -- → bool
```

---

### Input / Ввод

```lua
game.is_key_down(vk)                    -- → bool (held)
game.is_key_pressed(vk)                 -- → bool (rising edge)
game.key_just_pressed(vk)               -- → bool (edge detect, stateless)
game.key_just_released(vk)              -- → bool
game.key_held(vk)                       -- → bool
game.key_toggle(vk)                     -- → bool (toggle state)

game.mouse_click(button)                -- 0=left, 1=right
game.mouse_down(button)
game.mouse_up(button)
game.mouse_move(dx, dy)

game.send_key(vk)                       -- send keypress
```

---

### Math & Utilities / Математика и утилиты

```lua
game.lerp(a, b, t)                      -- → float
game.clamp(v, min, max)                 -- → float
game.dist2d(x1,y1, x2,y2)              -- → float
game.dist3d(x1,y1,z1, x2,y2,z2)        -- → float
game.normalize_angle(a)                 -- → float (-180..180)
game.angle_diff(a, b)                   -- → float
game.vec_angles(dx,dy,dz)               -- → pitch, yaw
game.fov_to_player(idx, bone)           -- → float (FOV angle)
game.angle_to_screen(pitch, yaw)        -- → sx, sy
game.sleep(ms)                          -- block for ms milliseconds / блокировать
game.log(...)                           -- print to console / вывод в консоль
```

---

### Cooldown Timers / Таймеры кулдауна

**EN:** Per-script named cooldown timers, identified by a string key.  
**RU:** Именованные таймеры кулдауна, идентифицируемые строкой.

```lua
game.cd_start("shoot", 150)             -- start 150 ms cooldown
game.cd_ready("shoot")                  -- → bool (expired?)
game.cd_elapsed("shoot")               -- → int (ms elapsed)
game.cd_remaining("shoot")             -- → int (ms remaining)
game.cd_reset("shoot")                 -- reset timer
```

---

### Weapon / Оружие

```lua
game.get_local_weapon_id()              -- → int (item definition index)
game.get_player_weapon_id(idx)          -- → int
game.get_local_ammo()                   -- → clip, reserve
game.get_player_ammo(idx)              -- → clip, reserve
game.is_local_reloading()               -- → bool
game.get_local_zoom_level()             -- → int
```

---

### Spread & Accuracy / Разброс и точность

```lua
game.get_local_spread()                 -- → float
game.get_player_spread(idx)            -- → float
game.get_local_inaccuracy()             -- → float
game.get_local_velocity_modifier()     -- → float
game.get_player_velocity_modifier(idx)  -- → float
```

---

### Steam IDs / Steam-идентификаторы

```lua
game.get_local_steamid()               -- → string (SteamID64)
game.get_player_steamid(idx)           -- → string
game.get_all_steamids()                -- → table {idx=steamid, ...}
```

---

### Colors / Цвета

```lua
game.get_local_health_ratio()          -- → float (0..1)
game.get_local_player_color()          -- → r, g, b (team-based color)
game.get_player_color(idx)             -- → r, g, b
```

---

## ui — UI Builder / Построитель UI

**EN:** Builds persistent UI panels (tabs and group boxes) that appear in the ForceSense menu. Call `ui.*` from `OnLoad()`.

**RU:** Строит постоянные UI-панели (вкладки и группы), появляющиеся в меню ForceSense. Вызывай из `OnLoad()`.

---

### Tabs / Вкладки

```lua
ui.add_tab("My Tab")                    -- create tab / создать вкладку
ui.remove_tab("My Tab")                 -- delete it / удалить
ui.rename_tab("My Tab", "New Name")
```

---

### Group Boxes / Группы

```lua
-- add_groupbox(tab, name [, x, y, w, h]) → bool
ui.add_groupbox("My Tab", "Options", 0, 0, 250, 200)
ui.remove_groupbox("My Tab", "Options")
ui.rename_groupbox("My Tab", "Options", "Settings")
```

---

### Elements / Элементы

```lua
-- add_checkbox(tab, group, id, label [, default]) → bool
ui.add_checkbox("My Tab", "Options", "chk_esp", "Enable ESP", false)

-- add_slider_float(tab, group, id, label, min, max [, default]) → bool
ui.add_slider_float("My Tab", "Options", "sld_fov", "FOV", 1, 180, 90)

-- add_slider_int(tab, group, id, label, min, max [, default]) → bool
ui.add_slider_int("My Tab", "Options", "sld_smooth", "Smooth", 1, 50, 10)

-- add_button(tab, group, id, label, callback) → bool
ui.add_button("My Tab", "Options", "btn_reload", "Reload", function()
    log.print("Button clicked!")
end)

-- add_separator(tab, group, id)
ui.add_separator("My Tab", "Options", "sep1")

-- remove_element(id)
ui.remove_element("chk_esp")
```

---

### Reading & Writing Values / Чтение и запись значений

```lua
-- get_value(id) → bool | float | int | nil
local enabled = ui.get_value("chk_esp")
local fov     = ui.get_value("sld_fov")

-- set_value(id, value)
ui.set_value("chk_esp", true)
ui.set_value("sld_fov", 60.0)
```

---

## imgui — ImGui Widgets

**EN:** Direct ImGui widget calls, available inside `OnMenuRender()`.  
**RU:** Прямые вызовы виджетов ImGui, доступны внутри `OnMenuRender()`.

```lua
imgui.text("Hello")
imgui.text_colored("Red text", {r=1,g=0,b=0,a=1})

-- changed, value = imgui.checkbox(label, value)
local ch, val = imgui.checkbox("Enable", true)

-- changed, value = imgui.slider_float(label, value, min, max)
local ch, val = imgui.slider_float("Speed", 1.0, 0.0, 10.0)

-- changed, value = imgui.slider_int(label, value, min, max)
local ch, val = imgui.slider_int("Count", 5, 1, 20)

-- clicked = imgui.button(label [, w, h])
if imgui.button("Click Me", 100, 24) then end

-- changed, text = imgui.input_text(label, text)
local ch, txt = imgui.input_text("Name", "default")

-- changed, color = imgui.color_edit3(label, {r,g,b,a})
local ch, col = imgui.color_edit3("Color", {r=1,g=0,b=0,a=1})

imgui.separator()
imgui.same_line()
imgui.spacing()

-- Containers
imgui.begin_child("id", 200, 100, true)
imgui.end_child()

-- Combo box
if imgui.begin_combo("Combo", "Current") then
    if imgui.selectable("Option A", false) then end
    imgui.end_combo()
end

-- Tab bar
if imgui.begin_tab_bar("tabs") then
    if imgui.begin_tab_item("Tab 1") then
        imgui.end_tab_item()
    end
    imgui.end_tab_bar()
end
```

---

## memory — Memory Read/Write / Чтение и запись памяти

**EN:** Safe-read wrappers for CS2 process memory. Returns `0` on access violations.  
**RU:** Безопасные функции чтения/записи памяти CS2. При ошибке доступа возвращают `0`.

```lua
memory.read_int(addr)        -- → int32
memory.read_uint(addr)       -- → uint32
memory.read_float(addr)      -- → float
memory.read_ptr(addr)        -- → uint64 (pointer)
memory.read_bool(addr)       -- → bool
memory.read_byte(addr)       -- → int (0..255)
memory.read_string(addr [,max_len])   -- → string (default max 64 bytes)

memory.write_int(addr, val)  -- → bool (success)
memory.write_float(addr, val)
memory.write_bool(addr, val)

-- Pointer chain: start at addr, dereference + add each offset
-- Цепочка указателей: разыменовать + прибавить каждое смещение
local result = memory.read_chain(base_addr, {0x10, 0x20, 0x8})

memory.is_attached()         -- → bool (CS2 process found)
```

---

## settings — Settings Access / Доступ к настройкам

**EN:** Read and write ForceSense internal settings by string key.  
**RU:** Чтение и запись внутренних настроек ForceSense по строковому ключу.

```lua
-- get(key) → value (bool, int, float, or color table)
local enabled = settings.get("esp_rendering")
local fov     = settings.get("aimbot_fov")
local col     = settings.get("enemy_color")  -- {r,g,b,a}

-- set(key, value)
settings.set("aimbot_fov", 15.0)
settings.set("aimbot_enabled", true)
settings.set("enemy_color", {r=1, g=0, b=0, a=1})

-- toggle(key) → new_value (bool settings only)
settings.toggle("esp_rendering")
```

**Bool keys:** `esp_rendering`, `esp_mode`, `esp_box_rendering`, `line_rendering`, `hp_bar_rendering`, `head_hitbox_rendering`, `skeleton_rendering`, `show_names`, `show_weapons`, `aimbot_enabled`, `trigger_bot_enabled`, `trigger_flash_check`, `trigger_scope_check`, `trigger_attack_all`, `seed_trigger_enabled`, `bhop_enabled`, `air_duck_enabled`, `show_speedometer`, `show_watermark`, `auto_accept_enabled`, `revolver_correction`, `recoil_overlay_enabled`, `show_bomb_info`, `show_bomb_esp`, `show_fov_radius`, `weapon_configs_enabled`

**Int keys:** `esp_type`, `aimbot_target`, `aimbot_scan_mode`, `trigger_bot_min_delay`, `trigger_bot_random_delay`, `seed_trigger_hitchance`, `seed_trigger_delay_ms`, `bind_aim_key`, `bind_trigger_key`, `bind_esp_key`

**Float keys:** `esp_box_size`, `skeleton_size`, `aimbot_fov`, `aimbot_smoothness`, `aimbot_range`, `seed_trigger_max_spread`, `recoil_overlay_thickness`

**Color keys:** `enemy_color`, `ally_color`, `head_hitbox_color`, `esp_box_color`, `skeleton_color`, `name_color`, `weapon_color`, `hp_bar_color`, `fov_radius_color`, `recoil_overlay_color`

---

## input — Input / Ввод

```lua
input.is_key_down(vk)        -- → bool (held)
input.is_key_pressed(vk)     -- → bool (rising edge, resets each call)
input.mouse_move(dx, dy)     -- relative mouse movement
input.key_constants()        -- → table of VK_* constants
```

**Key constants from `input.key_constants()` / Константы клавиш:**  
`VK_LBUTTON`, `VK_RBUTTON`, `VK_MBUTTON`, `VK_SHIFT`, `VK_CONTROL`, `VK_MENU`, `VK_SPACE`, `VK_INSERT`, `VK_DELETE`, `VK_F1`–`VK_F6`

---

## utils — Utilities / Утилиты

```lua
utils.world_to_screen(wx, wy, wz)   -- → sx, sy, visible
utils.dist2d(x1,y1, x2,y2)         -- → float
utils.dist3d(x1,y1,z1, x2,y2,z2)   -- → float
utils.is_cs2_active()               -- → bool (CS2 window in focus)
utils.get_fps()                     -- → float
utils.get_tick()                    -- → int (GetTickCount)
utils.screen_width()                -- → int
utils.screen_height()               -- → int
```

---

## client — Low-level CS2 Pointers

**EN:** Direct pointers into the CS2 client. For advanced use only.  
**RU:** Прямые указатели на данные клиента CS2. Только для продвинутого использования.

```lua
client.base()                        -- → int (client.dll base address)
client.local_pawn()                  -- → int (local player pawn pointer)
client.entity_list()                 -- → int (entity list address)
client.view_matrix()                 -- → table[16] (current view matrix)
client.offsets()                     -- → table of all known offset names/values
```

---

## log — Console Logging / Логирование

```lua
-- Print any number of values to the in-game console
-- Вывести любые значения в консоль
log.print("Hello", 42, true)

-- Clear the console log
-- Очистить консоль
log.clear()
```

---

## Complete Example / Полный пример

```lua
-- esp_example.lua
-- Draws a health bar above each enemy's head
-- Рисует полоску HP над головой каждого врага

local font_color = render.color(1, 1, 1, 1)
local bar_bg     = render.color(0.15, 0.15, 0.15, 0.85)
local bar_hp     = render.color(0.1, 0.8, 0.2, 1)

function OnLoad()
    log.print("esp_example loaded")
end

function OnFrame()
    local enemies = game.get_all_enemies()
    for _, idx in ipairs(enemies) do
        local sx, sy, vis = game.get_player_head_screen(idx)
        if vis then
            local hp  = game.get_player_hp(idx)
            local name = game.get_player_name(idx)
            local bw, bh = 40, 5
            local bx, by = sx - bw/2, sy - 14

            -- background bar
            render.rect_filled(bx, by, bw, bh, bar_bg, 2)
            -- HP fill
            render.rect_filled(bx, by, bw * (hp/100), bh, bar_hp, 2)
            -- name
            render.text(sx - 20, sy - 22, name, font_color, 11)
        end
    end
end

function OnUnload()
    log.print("esp_example unloaded")
end
```

---

*ForceSense Lua API Reference — generated from source headers*
