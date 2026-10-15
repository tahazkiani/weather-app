# Weather-App

A small web page that shows the current weather for a city, for people who want a simple weather app they can run and change on their own computer.

You type a city name and the page shows the temperature, a short description, humidity and wind speed. A button switches the temperature between °C and °F. It is plain HTML, CSS and JavaScript, so there is nothing to build. It gets its data from the [OpenWeatherMap](https://openweathermap.org/api) API, so you need your own free API key before it will show any weather.

This project has no `package.json` or `pyproject.toml`. The name `weather-app` is the name of this repository. There is no command called `weather-app`. You open `index.html` in a browser. A hosted copy is part of my portfolio at [tahak.fun/weather-app](https://tahak.fun/weather-app/).

## Table of Contents

- [Install](#install)
  - [Dependencies](#dependencies)
- [Usage](#usage)
  - [CLI](#cli)
- [Known limitations](#known-limitations)
- [API](#api)
- [Maintainers](#maintainers)
- [Credits](#credits)

## Install

You need `git` and a web browser. Python 3 is optional. It is only used to serve the folder on your computer. Check what you have:

```bash
git --version
python3 --version
```

I tested with git 2.50.1 and Python 3.14.3.

Download the project:

```bash
git clone https://github.com/tahazkiani/weather-app.git
cd weather-app
```

Create your config file from the template:

```bash
cp config.example.js config.js
```

Open `config.js` in a text editor and replace `YOUR_API_KEY_HERE` with your OpenWeatherMap API key. `config.js` is listed in `.gitignore`, so git will not commit your key. Do not remove that line.

To check the install worked, list the files:

```bash
ls -1 config.js index.html script.js style.css
```

You should see:

```text
config.js
index.html
script.js
style.css
```

If `ls` says `config.js` is missing, run the `cp` command again.

### Dependencies

Nothing to install by hand. The page loads Bootstrap 5.3.0-alpha3 from a CDN (cdn.jsdelivr.net), so it needs an internet connection. It also needs a connection to reach the OpenWeatherMap API.

You do need an API key. Sign up at <https://openweathermap.org/api>. A new key can take a while to start working. Until it does, you get the "Invalid API key" error shown in [Usage](#usage).

## Usage

The easiest way to start is to double-click `index.html`, or open it from your browser's File menu. The page works from a file. If you would rather serve the folder, the next section shows how.

Once the page is open, type a city name such as `Boston` and press **Get Weather**. The page fills in the city name, temperature, description, humidity, wind speed and an icon. Press **Switch to °F** to change units. The button only changes how the number is shown. It does not make a new request.

If you press **Get Weather** with the box empty, the page shows a pop-up that says `Please enter a city name.`

### CLI

The only things you type in a terminal are the commands to serve the folder. Run this from the project folder:

```bash
python3 -m http.server 8000
```

The terminal prints:

```text
Serving HTTP on :: port 8000 (http://[::]:8000/) ...
```

Go to <http://localhost:8000> in your browser. Each time the browser loads a file, the terminal prints a line like these:

```text
::1 - - [15/Oct/2026 12:11:56] "GET / HTTP/1.1" 200 -
::1 - - [15/Oct/2026 12:11:56] "GET /config.js HTTP/1.1" 200 -
```

Press `Ctrl+C` to stop the server. To use a different port, change `8000` in the command and in the address.

**When something goes wrong: missing config.** If you skip the `cp config.example.js config.js` step, the server logs a 404 for `config.js`:

```text
::1 - - [15/Oct/2026 12:11:56] code 404, message File not found
::1 - - [15/Oct/2026 12:11:56] "GET /config.js HTTP/1.1" 404 -
```

In the browser, pressing **Get Weather** then shows a red box:

```text
Error: No API key found. Copy config.example.js to config.js and add your key.
```

The fix is to run `cp config.example.js config.js`, add your key to the new file, and reload the page.

**When something goes wrong: wrong key.** If `config.js` still says `YOUR_API_KEY_HERE`, or the key is wrong or not active yet, the page shows the message OpenWeatherMap sends back:

```text
Error: Invalid API key. Please see https://openweathermap.org/faq#error401 for more info.
```

To check a key outside the page, put it in place of `YOUR_KEY` here. A working key returns weather data as JSON. A bad one returns a 401 message:

```bash
curl "https://api.openweathermap.org/data/2.5/weather?q=Boston&appid=YOUR_KEY&units=metric"
```

If your key is correct and the error stays, wait and try again. New keys can take a while to start working.

## Known limitations

- **The key is visible in the browser.** `config.js` is a file the browser downloads, so anyone who can open your copy of the page can read the key. That is fine on your own computer. If you put the page on the internet, use a key you can revoke, or move the API call to a small server.
- **Switch before search.** Pressing **Switch to °F** before searching for any city shows a meaningless number (`32.0°F`), because no temperature has been loaded yet. I found this by reading the code, not by running it.
- **Metric wind only.** Wind speed is always in m/s.
- **No forecast.** The page shows current conditions for one city.
- **Home button.** The "← Home" button links to `https://tahak.fun`. Change or remove it in `index.html` if you fork this project.
- **Online only.** The page does not work offline.

## API

This project is not a package, so there are no imports or exports. If you want to use the code from another script, `script.js` runs as a normal browser script. Its top-level functions are global, and so is the `window.WEATHER_API_KEY` setting that `config.js` creates. Load your own script after `script.js` and it can call or replace them.

### `window.WEATHER_API_KEY`

The OpenWeatherMap API key. Set it in `config.js`, which `index.html` loads before `script.js`.

- Type: string.
- Read once when `script.js` loads. Changing it afterwards has no effect until the page is reloaded.
- If it is not set, `getWeather` shows the "No API key found" error.

### `getWeather(city)`

Asks OpenWeatherMap for the current weather in `city` and shows it on the page. The **Get Weather** button calls this function.

- `city` (string): a city name. Spaces and special characters are encoded for you. It is not checked for emptiness here. The button handler does that.
- Returns: a `Promise` that resolves to `undefined` when it finishes.
- Throws: nothing. Errors are caught and shown in the red box on the page as `Error: <message>`. The cases are a missing key, a non-OK response from OpenWeatherMap (the message OpenWeatherMap sent, or `Unable to fetch weather data`), and a network failure.

```javascript
await getWeather('Boston');
```

### `displayWeather(data)`

Writes a weather response to the page: city name, temperature, description, humidity, wind speed and icon. `getWeather` calls it after a successful request.

- `data` (object): a response from the OpenWeatherMap "current weather" endpoint. These fields are read: `name`, `main.temp`, `main.humidity`, `weather[0].description`, `weather[0].icon` and `wind.speed`. `main.temp` must be in °C, which is what `getWeather` asks for.
- Returns: `undefined`.
- Throws: a `TypeError` if a field above is missing. When called by `getWeather`, this ends up in the red error box.

### `toggleTemperatureUnit()`

Switches the shown temperature between °C and °F. The **Switch to °F** button calls it.

- Takes no arguments.
- Returns: `undefined`.
- See [Known limitations](#known-limitations) for what happens if you call it before any weather is loaded.

### Internal

`updateTemperature()` and `hideErrorMessage()` are used by the functions above. The variables `currentTempCelsius` and `isCelsius` hold the page's state. Do not rely on any of these.

### Example: use it from your own page

Add a second script to `index.html`, after `script.js`, that calls `getWeather` on load:

```html
<script src="config.js"></script>
<script src="script.js"></script>
<script>
  document.getElementById('city-input').value = 'Boston';
  getWeather('Boston');
</script>
```

### Example: extend it

You can wrap `displayWeather` to run your own code each time weather loads. Save this as `extension.js` and load it after `script.js` in `index.html`:

```javascript
// extension.js
const originalDisplayWeather = window.displayWeather;

window.displayWeather = function (data) {
  originalDisplayWeather(data);
  console.log(`Loaded ${data.name}: ${data.main.temp}°C`);
};
```

```html
<script src="script.js"></script>
<script src="extension.js"></script>
```

`getWeather` looks up `displayWeather` each time it runs, so it picks up your version. Search for a city, open your browser's developer console, and you see a line like `Loaded Boston: 12.3°C` (the numbers will vary).

## Maintainers

[Taha Kiani](https://github.com/tahazkiani)

## Credits

- Original app, and this repository's code: Taha Kiani. The app was first built as part of [my portfolio website](https://tahak.fun).
- Claude (Anthropic's AI assistant) was used to split this project into its own repository, separate from the website, Taha created the README and the license.
- Weather data: [OpenWeatherMap](https://openweathermap.org/api).
- Styling: [Bootstrap](https://getbootstrap.com/).
- README structure: the [Standard Readme](https://github.com/RichardLitt/standard-readme) specification and the course README specification. The [HTTPie README](https://github.com/httpie/httpie) was my example of a good README.
- Markdown syntax: [GitHub's basic writing and formatting guide](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

