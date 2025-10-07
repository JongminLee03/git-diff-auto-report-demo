import git, datetime, os

repo = git.Repo(".")
today = datetime.date.today()
since = today - datetime.timedelta(days=1)

commits = list(repo.iter_commits(since=since.strftime("%Y-%m-%d")))
if not commits:
    print("No commits in the last 24 hours.")
    exit()

os.makedirs("reports", exist_ok=True)
report_path = f"reports/diff_report_{today}.md"

lines = [f"# 🧾 Daily Diff Report ({today})\n\n"]

for c in commits:
    lines.append(f"## Commit: {c.hexsha[:7]} by {c.author}")
    lines.append(f"**Date:** {c.committed_datetime}  \n**Message:** {c.message.strip()}\n")
    diffs = c.diff(c.parents[0] if c.parents else None, create_patch=True)
    for d in diffs:
        lines.append(f"**File:** {d.a_path or d.b_path}\n")
        lines.append("```diff")
        diff_text = d.diff.decode("utf-8", errors="ignore")
        lines.append(diff_text[:1000])  # 너무 길면 일부만
        lines.append("```\n")

with open(report_path, "w", encoding="utf-8") as f:
    f.write("\n".join(lines))

print(f"✅ Report saved: {report_path}")