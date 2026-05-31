# Developer Note

If you need to regenerate the skills, see [DESIGN.md](DESIGN.md) for the full
pipeline notes.

Build command:

```bash
deno run --allow-read --allow-write scripts/build.ts \
  --source ../requiems-api/apps/dashboard/config/api_docs \
  --output ./skills
```
