# deploy-legal.yml
  # 
  # Copy this file to .github/workflows/deploy-legal.yml in the servy-legal repo.
  # (Requires GitHub workflow scope to create via API — paste manually via GitHub UI)
  #
  ---
  name: Deploy Legal Docs

  on:
    push:
      branches: [main]
      paths:
        - '*.pdf'

  jobs:
    deploy:
      name: Deploy PDFs to Vercel public folder
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4

        - name: Trigger Servy platform rebuild
          run: |
            curl -X POST \
              -H "Authorization: Bearer ${{ secrets.SERVY_GITHUB_TOKEN }}" \
              -H "Accept: application/vnd.github.v3+json" \
              "https://api.github.com/repos/MostGo1/servy-platform/actions/workflows/deploy.yml/dispatches" \
              -d '{"ref":"main"}'
            echo "Triggered servy-platform rebuild"

        - name: Notify users of legal update
          run: |
            curl -X POST \
              "${{ secrets.SUPABASE_FUNCTION_URL }}/notify-legal-update" \
              -H "Authorization: Bearer ${{ secrets.SUPABASE_SERVICE_KEY }}" \
              -H "Content-Type: application/json" \
              -d "{\"document\": \"$(git diff --name-only HEAD~1 HEAD | grep pdf | head -1)\", \"version\": \"$(date +%Y-%m)\", \"commit\": \"$GITHUB_COMMIT_MESSAGE\"}"
            echo "User notifications sent"
          env:
            GITHUB_COMMIT_MESSAGE: ${{ github.event.head_commit.message }}
  