"""
Workplace Incident Report Logger
Author: Liban Mohamed
Description: A command-line tool that logs workplace safety incidents
             to a CSV file for recordkeeping and review.
"""

import csv
import os
from datetime import datetime


FILENAME = "incident_reports.csv"
FIELDS = ["ID", "Date", "Time", "Reporter", "Location", "Incident Type", "Description", "Severity", "Reported At"]

INCIDENT_TYPES = [
    "Slip/Trip/Fall",
    "Equipment Malfunction",
    "Chemical Exposure",
    "Fire/Smoke",
    "Injury",
    "Near Miss",
    "Property Damage",
    "Other"
]

SEVERITY_LEVELS = ["Low", "Medium", "High", "Critical"]


def setup_file():
    """Create the CSV file with headers if it doesn't exist."""
    if not os.path.exists(FILENAME):
        with open(FILENAME, mode="w", newline="") as f:
            writer = csv.DictWriter(f, fieldnames=FIELDS)
            writer.writeheader()
        print(f"[+] Created new incident log: {FILENAME}\n")


def get_next_id():
    """Return the next available incident ID."""
    try:
        with open(FILENAME, mode="r") as f:
            rows = list(csv.DictReader(f))
            return len(rows) + 1
    except FileNotFoundError:
        return 1


def choose_from_list(prompt, options):
    """Display a numbered menu and return the user's choice."""
    print(f"\n{prompt}")
    for i, option in enumerate(options, 1):
        print(f"  {i}. {option}")
    while True:
        try:
            choice = int(input("Enter number: "))
            if 1 <= choice <= len(options):
                return options[choice - 1]
            else:
                print(f"Please enter a number between 1 and {len(options)}.")
        except ValueError:
            print("Invalid input. Please enter a number.")


def log_incident():
    """Collect incident details from the user and save to CSV."""
    print("\n--- NEW INCIDENT REPORT ---")

    now = datetime.now()
    incident_id = get_next_id()

    date = input("Date of incident (YYYY-MM-DD) [press Enter for today]: ").strip()
    if not date:
        date = now.strftime("%Y-%m-%d")

    time = input("Time of incident (HH:MM) [press Enter for now]: ").strip()
    if not time:
        time = now.strftime("%H:%M")

    reporter = input("Your name: ").strip()
    location = input("Location of incident (e.g. Warehouse Floor B): ").strip()
    incident_type = choose_from_list("Type of incident:", INCIDENT_TYPES)
    description = input("Brief description of what happened: ").strip()
    severity = choose_from_list("Severity level:", SEVERITY_LEVELS)

    report = {
        "ID": incident_id,
        "Date": date,
        "Time": time,
        "Reporter": reporter,
        "Location": location,
        "Incident Type": incident_type,
        "Description": description,
        "Severity": severity,
        "Reported At": now.strftime("%Y-%m-%d %H:%M:%S")
    }

    with open(FILENAME, mode="a", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=FIELDS)
        writer.writerow(report)

    print(f"\n[✓] Incident #{incident_id} logged successfully.")
    if severity in ["High", "Critical"]:
        print(f"[!] ALERT: This is a {severity.upper()} severity incident. Notify your supervisor immediately.")


def view_reports():
    """Display all logged incidents in a readable format."""
    try:
        with open(FILENAME, mode="r") as f:
            rows = list(csv.DictReader(f))

        if not rows:
            print("\nNo incidents have been logged yet.")
            return

        print(f"\n--- INCIDENT LOG ({len(rows)} total) ---")
        for row in rows:
            print(f"\n  ID:          {row['ID']}")
            print(f"  Date/Time:   {row['Date']} at {row['Time']}")
            print(f"  Reporter:    {row['Reporter']}")
            print(f"  Location:    {row['Location']}")
            print(f"  Type:        {row['Incident Type']}")
            print(f"  Severity:    {row['Severity']}")
            print(f"  Description: {row['Description']}")
            print(f"  Logged At:   {row['Reported At']}")
            print("  " + "-" * 40)

    except FileNotFoundError:
        print("\nNo incident log file found. Log an incident first.")


def summary():
    """Show a quick summary of incidents by severity."""
    try:
        with open(FILENAME, mode="r") as f:
            rows = list(csv.DictReader(f))

        if not rows:
            print("\nNo incidents to summarize.")
            return

        counts = {level: 0 for level in SEVERITY_LEVELS}
        for row in rows:
            if row["Severity"] in counts:
                counts[row["Severity"]] += 1

        print(f"\n--- SUMMARY ({len(rows)} total incidents) ---")
        for level, count in counts.items():
            bar = "█" * count
            print(f"  {level:<10} {bar} ({count})")

    except FileNotFoundError:
        print("\nNo incident log file found.")


def main():
    setup_file()
    print("========================================")
    print("   Workplace Incident Report Logger")
    print("========================================")

    while True:
        print("\nWhat would you like to do?")
        print("  1. Log a new incident")
        print("  2. View all incidents")
        print("  3. View summary")
        print("  4. Exit")

        choice = input("\nEnter choice (1-4): ").strip()

        if choice == "1":
            log_incident()
        elif choice == "2":
            view_reports()
        elif choice == "3":
            summary()
        elif choice == "4":
            print("\nExiting. Stay safe out there.\n")
            break
        else:
            print("Invalid choice. Please enter 1, 2, 3, or 4.")


if __name__ == "__main__":
    main()
