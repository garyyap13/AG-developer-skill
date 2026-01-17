# Setup Git Repository

Follow these steps to upload to GitHub:

## 1. Navigate to the folder
```powershell
cd d:\OHCRM_AG\ohcrm-developer-skill
```

## 2. Initialize Git
```bash
git init
git add .
git commit -m "Initial commit: OHCRM Developer Skill for Antigravity"
```

## 3. Create GitHub Repository
1. Go to: https://github.com/new
2. Repository name: `ohcrm-developer-skill`
3. Description: `Antigravity skill for OHCRM development with cross-module business logic awareness`
4. Choose: Public ✅ (so others can use it)
5. **Don't** check "Add README" (you already have one)
6. Click "Create repository"

## 4. Push to GitHub
Copy the commands GitHub shows you, or use these:

```bash
git remote add origin https://github.com/YOUR_USERNAME/ohcrm-developer-skill.git
git branch -M main
git push -u origin main
```

Replace `YOUR_USERNAME` with your GitHub username.

## 5. Share the Link!
Once pushed, your skill will be at:
```
https://github.com/YOUR_USERNAME/ohcrm-developer-skill
```

Others can install it with:
```bash
cd their-project/.agent/skills
git clone https://github.com/YOUR_USERNAME/ohcrm-developer-skill.git ohcrm-developer
```

## Success!
Your skill is now shareable online! 🎉
