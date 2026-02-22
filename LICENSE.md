## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

## Disclaimer

This repository contains local AI automation setup examples using OpenClaw, Ollama, and third-party messaging integrations (e.g., Telegram).  
Use at your own risk. The maintainers are not responsible for any misuse, data loss, or unintended automation behaviour.

Always review security settings before exposing any agent to external networks or messaging platforms.

## Security Notes

- Do not commit API tokens or secrets
- Keep `.openclaw/openclaw.json` out of version control
- Prefer environment variables for sensitive credentials
- Run periodic security audits:

```bash
openclaw security audit --deep
