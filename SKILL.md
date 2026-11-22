import math

start = 10_000        # starting money
target = 1_000_000_000
for rate in (0.07, 0.10, 0.20, 0.30):
    years = math.log(target / start) / math.log(1 + rate)
    print(f"{rate:.0%} yearly return: {years:.0f} years")

import argparse


def years_to_goal(goal, monthly, rate, start=0.0, max_months=1200):
    """Return years to reach goal, or None if not reached within max_months."""
    bal, months = start, 0
    while bal < goal and months < max_months:
        bal = bal * (1 + rate / 12) + monthly
        months += 1
    return months / 12 if bal >= goal else None


def main():
    p = argparse.ArgumentParser(description="Years to reach a savings goal.")
    p.add_argument("--goal", type=float, required=True)
    p.add_argument("--monthly", type=float, default=0.0, help="monthly contribution")
    p.add_argument("--rate", type=float, default=0.07, help="yearly return, e.g. 0.07")
    p.add_argument("--start", type=float, default=0.0, help="starting balance")
    a = p.parse_args()

    years = years_to_goal(a.goal, a.monthly, a.rate, a.start)
    if years is None:
        print("Goal not reached within 100 years with these numbers.")
    else:
        print(f"About {years:.1f} years to reach {a.goal:,.0f}.")


if __name__ == "__main__":
    main()


import sys


def main():
    if len(sys.argv) < 2:
        print("Usage: python leak_finder.py Name=monthly_cost [Name=monthly_cost ...]")
        sys.exit(1)

    expenses = {}
    for arg in sys.argv[1:]:
        name, sep, cost = arg.partition("=")
        if not sep:
            print(f"Skipping '{arg}': use Name=amount")
            continue
        try:
            expenses[name] = float(cost)
        except ValueError:
            print(f"Skipping '{arg}': amount must be a number")

    if not expenses:
        sys.exit(1)

    for name, cost in sorted(expenses.items(), key=lambda x: -x[1]):
        print(f"{name}: {cost * 12:,.0f}/year")
    print(f"Total: {sum(expenses.values()) * 12:,.0f}/year")


if __name__ == "__main__":
    main()

import argparse
import random


def run(years, start, mean, stdev):
    bal = start
    for _ in range(years):
        bal = max(bal * random.gauss(mean, stdev), 0)
    return bal


def main():
    p = argparse.ArgumentParser(description="Odds of reaching a target by investing alone.")
    p.add_argument("--target", type=float, default=1e9)
    p.add_argument("--years", type=int, default=40)
    p.add_argument("--start", type=float, default=10_000)
    p.add_argument("--mean", type=float, default=1.10, help="average yearly growth factor")
    p.add_argument("--stdev", type=float, default=0.25, help="yearly volatility")
    p.add_argument("--trials", type=int, default=100_000)
    a = p.parse_args()

    results = sorted(run(a.years, a.start, a.mean, a.stdev) for _ in range(a.trials))
    hit = sum(r >= a.target for r in results) / a.trials
    print(f"Chance of reaching {a.target:,.0f}: {hit:.4%}")
    print(f"Median outcome after {a.years} years: {results[a.trials // 2]:,.0f}")


if __name__ == "__main__":
    main()

