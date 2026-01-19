# afm_prototype.py

import re
from collections import defaultdict

EVASIVE_PATTERNS = [
    r"as an ai",
    r"i cannot",
    r"i'm unable",
    r"i don't have access",
    r"i cannot provide"
]

def detect_evasive_language(text):
    text = text.lower()
    return any(re.search(p, text) for p in EVASIVE_PATTERNS)

def detect_low_quality(prompt, response):
    return len(response.split()) < 20

def detect_repetition(response):
    words = response.lower().split()
    return len(words) - len(set(words)) > 10

def detect_contradiction(prompt, response):
    return "yes" in response.lower() and "no" in response.lower()

def analyze_conversation(prompt, response):
    issues = []

    if detect_evasive_language(response):
        issues.append("evasive_response")

    if detect_low_quality(prompt, response):
        issues.append("low_quality")

    if detect_repetition(response):
        issues.append("repetition")

    if detect_contradiction(prompt, response):
        issues.append("contradiction")

    return issues

def analyze_dataset(conversations):
    report = defaultdict(list)

    for idx, convo in enumerate(conversations):
        issues = analyze_conversation(convo["prompt"], convo["response"])
        for issue in issues:
            report[issue].append(idx)

    return report

if __name__ == "__main__":
    conversations = [
        {
            "prompt": "Explain quantum computing simply",
            "response": "As an AI, I cannot explain that in detail."
        },
        {
            "prompt": "Is the sky blue?",
            "response": "Yes. No. It depends."
        },
        {
            "prompt": "Write a poem",
            "response": "Poem poem poem poem poem poem poem poem poem"
        }
    ]

    results = analyze_dataset(conversations)

    print("=== AUTOMATIC FAILURE MINING REPORT ===")
    for issue, cases in results.items():
        print(f"{issue}: found in conversations {cases}")
