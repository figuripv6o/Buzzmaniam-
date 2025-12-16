# Buzzmaniam-

BUZZFLOW — ONE SHOT AUTO WORKFLOW
AUTO • PUSH • RUN • MERGE
PROJECT: Buzzmaniam
================================

# ---------- LOCAL: one-shot runner ----------
# save as: buzzflow.sh
#!/bin/bash
set -e

REPO="buzzmaniam"
BRANCH="main"
MSG="BuzzFlow auto run"

echo "[BuzzFlow] run pipeline"
python3 android/artifact_scan.py
python3 signals/signal_log.py
python3 signals/detect_anomaly.py
python3 core/regroup.py
python3 core/policy_gate.py

echo "[BuzzFlow] commit + push"
git add .
git commit -m "$MSG" || true
git push origin "$BRANCH"

echo "[BuzzFlow] done"


# ---------- LOCAL: init + first push (one time) ----------
git init
git branch -M main
git remote add origin https://github.com/YOURNAME/buzzmaniam.git
bash buzzflow.sh


# ---------- CI: GitHub Actions ----------
# .github/workflows/buzzflow.yml
name: BuzzFlow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  buzzflow:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.10"

      - name: Run BuzzFlow pipeline
        run: |
          python android/artifact_scan.py
          python signals/signal_log.py
          python signals/detect_anomaly.py
          python core/regroup.py
          python core/policy_gate.py

      - name: Upload report
        uses: actions/upload-artifact@v4
        with:
          name: buzzflow-report
          path: data/final_report.json


# ---------- AUTO-MERGE (optional, repo setting) ----------
# Enable in GitHub:
# Settings → Pull Requests → Allow auto-merge
# Branch protection: require BuzzFlow check


# ---------- RUN ----------
bash buzzflow.sh

END — BUZZFLOW ONE SHOT
