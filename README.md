# DupeSkin

**Take a selfie, learn what your skin needs, and find budget-friendly dupes for the products you already use. Then check after 7 days whether the swap actually worked on your skin.**

Built for the [YouCam API Skin AI & eCommerce VTO Hackathon](https://youcam-api-skin-ai-ecommerce-vto-hackathon.devpost.com/) (topic: Skin AI).

> Status: work in progress. Items marked **[planned]** are not built yet; update this section before submitting.

## The problem

People buy skincare based on ads, price tags, and brand names, without knowing what their skin actually needs. The result is wasted money, and sometimes irritated skin. The question at the moment of purchase is simple: *"Is this product right for my skin, and is there a cheaper one that does a similar job?"*

## How it works

1. **Selfie.** The user takes or uploads a photo.
2. **Skin analysis.** The YouCam Skin AI API scores the skin on concerns such as moisture, oiliness, pores, age spots, redness, texture, wrinkles, and radiance.
3. **Dupe matching.** The backend maps the user's needs to key active ingredients and compares the product they use today with cheaper products that share the same key actives, ranked by ingredient overlap and unit price.
4. **7-day check [planned].** After a week the user takes a second selfie. The app compares the raw scores and shows the measured change on their skin.

Scores for the 7-day check are stored on the device only. Photos are not stored.

## How the YouCam API is used

DupeSkin uses the **YouCam AI Skin Analysis API** (SD mode, 8 concerns per analysis):

| Step | Request | Purpose |
|---|---|---|
| 1 | `POST /file` | Register the selfie and receive a pre-signed upload URL |
| 2 | `PUT <upload url>` | Upload the image bytes |
| 3 | `POST /task/skin-analysis` | Create the analysis task (`moisture`, `oiliness`, `pore`, `age_spot`, `redness`, `texture`, `wrinkle`, `radiance`) |
| 4 | `GET /task/skin-analysis/{task_id}` | Poll until the task succeeds, then read `data.results.output` |

The backend uses each concern's `raw_score` (unadjusted, 1-100) rather than `ui_score`, so scores from two different days can be compared fairly. The API key stays on the server and never reaches the client.

## Architecture

```
Flutter app (web / iOS)  ->  Flask REST API  ->  YouCam Skin AI API
                                  |
                                  +-> products.json (dupe data)
```

```
dupeskin/
├── README.md
├── LICENSE
├── backend/        Flask API (app.py, api.py, dupes.py, products.json)
└── frontend/       Flutter app
```

## Setup

### Requirements

- Python 3.9+
- [Flutter](https://docs.flutter.dev/get-started) 3.x
- A YouCam API key (see the hackathon page for free API units)

### Backend

```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env    # then add your YOUCAM_API_KEY to .env
python app.py
```

The API runs at `http://localhost:5001`.

### Frontend

```bash
cd frontend
flutter pub get
flutter run -d web-server --web-port 8080
```

Open `http://localhost:8080` in your browser. Chrome users can use `flutter run -d chrome` instead.

## Data

`backend/products.json` is a small, hand-compiled dataset of skincare products (brand, price, size, key actives, ingredient list, source URL). Prices change, so treat them as examples from the date of compilation. Real-time pricing is out of scope for this prototype.

## Limitations

- The product dataset is small and hand-made.
- A dupe shares similar key ingredients with the original; concentration and formulation can differ.
- The 7-day comparison depends on similar lighting and angle between the two selfies. Results are measurements of the app's analysis, not clinical proof.
- 7-day history lives on one device and is lost if browser or app data is cleared. There are no user accounts by design.

## Privacy

- Selfies are sent to the YouCam API for analysis and are not saved by DupeSkin.
- No user accounts, no server-side user data.
- Check YouCam's own data retention terms for how long they keep analysis results (the documentation states results are kept for 24 hours after completion).

## Disclaimer

DupeSkin is for information only and is not medical advice. It does not diagnose skin conditions. See a dermatologist for skin concerns.

## License

[MIT](LICENSE)
