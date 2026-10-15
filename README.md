# Weather App

A small weather lookup page. Type a city, get the temperature, a short description, humidity and wind speed. You can switch between °C and °F.

It's plain HTML, CSS and JavaScript. There's nothing to install or build. It uses the [OpenWeatherMap](https://openweathermap.org/api) API for data and Bootstrap (from a CDN) for styling.

A hosted copy is part of my portfolio at [tahak.fun/weather-app](https://tahak.fun/weather-app/).

## Run it locally

You need a free OpenWeatherMap API key. Sign up at <https://openweathermap.org/api>. New keys can take a little while to start working.

1. Fork or clone this repo:

   ```bash
   git clone https://github.com/tahazkiani/weather-app.git
   cd weather-app
   ```

2. Create your config file and add your key:

   ```bash
   cp config.example.js config.js
   ```

   Then open `config.js` and replace `YOUR_API_KEY_HERE` with your key.

3. Open `index.html` in your browser. Or serve the folder, which is closer to how a real site behaves:

   ```bash
   python3 -m http.server 8000
   ```

   Then go to <http://localhost:8000>.

`config.js` is in `.gitignore`, so your key won't get committed. Don't remove that line.

## Files

| File | What it is |
| --- | --- |
| `index.html` | The page. |
| `script.js` | Fetches the weather and updates the page. |
| `style.css` | A few style tweaks on top of Bootstrap. |
| `config.example.js` | Template for your API key. |

## Known limits

- The key sits in a JavaScript file that the browser downloads, so anyone using your copy of the site can see it. That's fine for running locally. If you host it publicly, use a key you can revoke, or move the API call behind a small server.
- The wind speed shows in m/s only.
- The "← Home" button links to my site. Change or remove it in `index.html` if you fork this.

## License

[MIT](LICENSE)
