
# PhisX - Chrome Extension for detection of phishing URLs

## Problem Statement

Phishing attacks have become increasingly sophisticated and prevalent, posing a significant threat
to individuals and organizations. Cybercriminals create fraudulent websites and URLs that appear
legitimate, tricking users into revealing sensitive information or installing malware. This project aims to develop a Chrome extension that can effectively
detect and protect users from phishing URLs in real-time, providing an additional layer of security
against these malicious attempts.


## Project

PhisX is a Chrome extension designed to detect and block phishing URLs in real-time, enhancing online security against sophisticated cyber threats.


## Solution

**Choosing a Dataset**:
   - Identify and select a comprehensive dataset containing labeled URLs, distinguishing between legitimate and phishing URLs.
   
**Training a Model**:
   - Preprocess the dataset to prepare it for training.
   - Implement and train machine learning classification algorithms to accurately distinguish between phishing and legitimate URLs.
   - Evaluate and fine-tune the model to ensure high accuracy and reliability.

**Creating an API**:
   - Develop a RESTful API using Flask to serve the trained machine learning model.
   - Ensure the API can receive URL input, process it using the ML model, and return the detection results in real-time.

**Developing the Chrome Extension**:
   - Design and implement a Chrome extension that interacts with the Flask API.
   - Ensure the extension can send URL data to the API and display alerts or block access when a phishing URL is detected.
   - Integrate user-friendly features for seamless interaction and effective real-time protection.


## Tech Stack

1. Dataset: https://archive.ics.uci.edu/dataset/327/phishing+websites

2. ML classification algorithms

3. Flask for API

4. Docker for containerization, Gunicorn as the production server

5. React + TypeScript + TailwindCSS + AceternityUI - Chrome extension

6. CI/CD - GitHub Actions builds the API image and pushes it to Docker Hub

7. Deployment - Render runs the image from Docker Hub --> [https://phisx-api-latest.onrender.com/health](https://phisx-api-latest.onrender.com/health)


## Project Structure

```
Flask API/          Flask API: /predict endpoint, feature extraction (inputScript.py), trained model (.pkl), Dockerfile
chrome_extension/   Chrome extension popup (React + Vite, Manifest V3)
model training/     UCI dataset and the Jupyter notebook used to train and compare models
evaluation/         Offline and end-to-end evaluation scripts, results and figures
.github/workflows/  CI pipeline that builds, pushes and deploys the API
```


## API

| Method | Route      | Body                          | Response |
| ------ | ---------- | ----------------------------- | -------- |
| POST   | `/predict` | `{ "url": "https://..." }`    | `{ "url", "prediction": "Phishing Website" \| "Legitimate website", "path": "heuristic" \| "model" }` |
| GET    | `/health`  | -                             | `{ "status": "ok" }` |

`path` tells you how the result was produced. `heuristic` means an obvious phishing sign (IP address as hostname, `@` in the URL, or a URL shortener) was found and the model was skipped. `model` means the 30 features were extracted and passed to the Logistic Regression model. The model path fetches the page and runs WHOIS lookups, so it usually takes 5-15 seconds.


## Running Locally

**Requirements:** Python 3.11+ (tested on 3.13) and Node.js 20.19+.

### 1. Start the API

```sh
cd "Flask API"
python -m venv myenv
source myenv/bin/activate        # Windows: myenv\Scripts\activate
pip install -r requirements.txt
python app.py
```

The API runs at `http://127.0.0.1:5000`. Test it:

```sh
curl -X POST http://127.0.0.1:5000/predict -H "Content-Type: application/json" -d '{"url": "https://www.wikipedia.org/"}'
```

Or run it with Docker (same image as production):

```sh
cd "Flask API"
docker build -t phisx-api .
docker run -p 5000:5000 phisx-api
```

### 2. Build and load the extension

```sh
cd chrome_extension
npm ci
npm run build
```

1. Open `chrome://extensions` and turn on **Developer mode**.
2. Click **Load unpacked** and select the `chrome_extension/dist` folder.
3. Open any website and click the PhisX icon.

By default the extension calls `http://127.0.0.1:5000`. To use a deployed API instead, create `chrome_extension/.env.production` before building:

```
VITE_API_URL=https://your-app.onrender.com
```

After any code change, run `npm run build` again and click the reload icon on the PhisX card in `chrome://extensions`.


## Deployment

The API is deployed to Render as a prebuilt Docker image. Render does not build the code itself.

```
git push (changes in Flask API/)
  -> GitHub Actions builds the image
  -> pushes it to Docker Hub as :latest and :<commit-sha>
  -> calls the Render deploy hook with the :<commit-sha> image
  -> Render pulls that image and runs it
```

The pipeline is defined in [`.github/workflows/deploy-api.yml`](.github/workflows/deploy-api.yml). It only runs when files in `Flask API/` (or the workflow itself) change, and it can also be started manually from the **Actions** tab. Each deploy uses the exact image built for that commit, so rolling back means redeploying an older `:<commit-sha>` tag.

### One-time setup

1. **Docker Hub:** create an access token with Read & Write permission (Account settings -> Personal access tokens).
2. **GitHub:** in **Settings -> Secrets and variables -> Actions**, add these as **Repository secrets** (not Environment secrets, which the workflow can't read):
   - `DOCKERHUB_USERNAME`: your Docker Hub username
   - `DOCKERHUB_TOKEN`: the token from step 1
3. **Push to `main`** so the workflow builds and pushes the first image. The Render step is skipped until its secret exists.
4. **Render:** choose **New -> Web Service -> Existing Image** and use `docker.io/<your-username>/phisx-api:latest`. Pick an instance type and set **Health Check Path** to `/health`. No environment variables are needed, because Render sets `PORT` itself.
5. **Deploy hook:** in the Render service, copy the URL from **Settings -> Deploy Hook** and add it as a GitHub secret named `RENDER_DEPLOY_HOOK_URL`.

From then on, every push that changes the API is deployed automatically.

If the Docker Hub repository is private, also add a registry credential in Render so it can pull the image.

### Cold starts on the free plan

Render's free plan puts the service to sleep after 15 minutes without requests. The next request wakes it, which takes about 30-60 seconds, and the extension shows "Loading..." until then. To keep it awake, have an uptime monitor (for example UptimeRobot or cron-job.org) call `GET /health` every 10-14 minutes.

## Evaluation (dissertation results)

Offline metrics, LR vs RF trade-off, controlled HTTP fixtures, and pipeline latency:

See `evaluation/README.md` and `evaluation/DISSERTATION_CORRECTIONS.md`.


## Work flow

![image](https://github.com/user-attachments/assets/688d1970-b97f-4b23-a9b5-c2ce78a97374)

## Check out this video

[![Watch this video on YouTube](https://img.youtube.com/vi/CAlDwQ-C4pI/0.jpg)](https://youtu.be/CAlDwQ-C4pI?si=s-igmY8i9POim0QD)


## Author

- [@saiabhiramjaini](https://github.com/saiabhiramjaini)

- Repo - https://github.com/saiabhiramjaini/PhisX-Chrome-Extension

## Feedback

If you have any feedback, please reach out me at abhiramjaini28@gmail.com


## Publishing a New Extension Version

1. Bump `version` in both `chrome_extension/package.json` and `chrome_extension/public/manifest.json`.

2. Make sure `chrome_extension/.env.production` points to the deployed API, then build the extension:
   ```sh
   cd chrome_extension
   npm run build
   ```
   This will generate a `dist` folder with the production build.

3. Create a ZIP file of the contents of the `dist` folder:
   ```sh
   cd dist
   zip -r ../chrome-extension.zip .
   cd ..
   ```

4. Upload the generated `chrome-extension.zip` file to the Chrome Web Store Developer Dashboard.