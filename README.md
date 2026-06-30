# SAP Clean Core Workshop (Kiro skills + agent)

This repo is a ready-to-run [Kiro CLI](https://kiro.dev/cli) workflow for checking an ABAP package against SAP Clean Core and fixing what it finds. It scores every object on SAP's A to D compliance levels and walks you through assess, prep, fix, and review. Under the hood it drives [`abapctl`](https://github.com/aws-for-sap/Automate-SAP-development-workflows-using-ABAP-CTL), the headless ABAP CLI, over ADT.

You don't need to assemble anything. The skills, the worker agent, and the reference data (the `cc-kb/` style guidance plus the SAP cloudification catalog) all live here. Clone it into your project, point it at your SAP system, and you're going.

```
assess  ->  prep  ->  fix  ->  review      (Kiro skills + agent)
        |
        v
abapctl     the ABAP development CLI it runs on
        |   HTTPS / ADT REST
        v
SAP         your ECC / S/4HANA system
```

## What's in here

```
.kiro/
  agents/
    clean-core-remediator.json          worker agent (tools, model, config)
    clean-core-remediator.prompt.md     worker agent system prompt
  skills/
    sap-abap/                abapctl command reference the agents read
    clean-core-assess/       ATC run + A/B/C/D classification (read-only)
    clean-core-prep/         download source, baselines, findings; resolve transport
    clean-core-fix/          spawn remediators in parallel
    clean-core-review/       retrospective + exemption candidates
.abapctl/
  reference/
    cc-kb/                   remediation style/idiom guidance (read by the agent)
    sap-api-reference/       SAP released-API + classification catalog (JSON)
.abapctl.json.template       copy to .abapctl.json and fill in your connection
```

## Before you start

You'll need a few things in place:

- [`abapctl`](https://github.com/aws-for-sap/Automate-SAP-development-workflows-using-ABAP-CTL) on your `PATH` (check with `abapctl --version`). Grab the latest release.
- [Kiro CLI](https://kiro.dev/cli) installed.
- Subagents turned on, since the fix step spawns remediators: `kiro-cli settings chat.enableSubagent true`.
- A SAP system you can reach with the ADT services exposed.

## Setup

First, get the files into your project. You can either work straight inside a clone of this repo, or copy the two trees into a project you already have. Those trees are the skills and agent (`.kiro/`) and the reference data (`.abapctl/reference/`, which carries both `cc-kb/` and `sap-api-reference/`):

```bash
cp -r .kiro <your-project>/
mkdir -p <your-project>/.abapctl
cp -r .abapctl/reference <your-project>/.abapctl/
```

That puts the knowledge base at `<your-project>/.abapctl/reference/cc-kb/` and the SAP catalog at `<your-project>/.abapctl/reference/sap-api-reference/`, which is exactly where the agent and `abapctl` look for them.

Next, make your connection config from the template and fill in your host, SID, client, and user:

```bash
cp .abapctl.json.template .abapctl.json
# edit .abapctl.json
```

Then put the SAP password in the environment variable named by `password_env` (it defaults to `ABAPCTL_PASSWORD`):

```bash
export ABAPCTL_PASSWORD='your-sap-password'
```

And confirm you can reach the system:

```bash
abapctl system-check -c dev
```

The bundled `.abapctl/reference/` works as-is, so there's nothing to download. If you want to freshen the SAP catalog later, run `abapctl reference update`.

## Using AWS Secrets Manager for the SAP password

If you'd rather not keep the password in an environment variable, a connection can pull it from AWS Secrets Manager instead. Set `password_aws_secret` (the secret id) on the connection in `.abapctl.json`, in place of `password_env`:

```json
{
  "connections": {
    "dev": {
      "host": "your-sap-host.example.com",
      "sid": "S4D",
      "client": "100",
      "secure": true,
      "auth": "basic",
      "username": "YOUR_SAP_USER",
      "password_aws_secret": "my/sap/dev-password",
      "language": "EN"
    }
  }
}
```

One thing to know up front: this needs the AWS CLI installed and configured. `abapctl` shells out to the `aws` command rather than bundling an AWS SDK, so the machine running the workflow has to have:

- the AWS CLI v2 installed and on `PATH` (check with `aws --version`), and
- working credentials for it (from `aws configure`, an SSO profile, environment variables, or an instance role) that are allowed to read the secret. You can confirm both with `aws sts get-caller-identity` and, if you want to be sure, `aws secretsmanager get-secret-value --secret-id my/sap/dev-password`.

Region, profile, and credentials all come from the `aws` CLI's own configuration. `abapctl` passes none of its own.

A few details on how the secret is read:

- A plain-string secret is used as the password verbatim. A JSON object with a single key uses that key's value. A JSON object with several keys needs `password_aws_key` to say which field holds the password.
- `password_aws_secret` wins over `password_env`. If a configured Secrets Manager fetch fails, it errors out rather than quietly falling back to the environment variable.
- The secret value never lands in logs, error messages, or `--json` output.

When you go this route you don't export `ABAPCTL_PASSWORD` at all. Skip that step and head straight to `abapctl system-check -c dev`.

## References

- [`abapctl`](https://github.com/aws-for-sap/Automate-SAP-development-workflows-using-ABAP-CTL), the headless ABAP development CLI this workflow runs on.
- [SAP Clean Core Extensibility whitepaper](https://www.sap.com/documents/2024/09/20aece06-d87e-0010-bca6-c68f7e60039b.html), which explains the four compliance levels.
- [SAP/abap-atc-cr-cv-s4hc](https://github.com/SAP/abap-atc-cr-cv-s4hc), the cloudification catalog (Apache 2.0) bundled under `sap-api-reference/`.
- [SAP/styleguides](https://github.com/SAP/styleguides), the Clean ABAP guidance (Apache 2.0).

## License

This repository's own files (the skills, agent, README, and templates) are MIT-0, MIT No Attribution. See [LICENSE](LICENSE).

It also bundles SAP reference material (the `cc-kb/` guidance and the `sap-api-reference/` catalog) that stays under its original Apache License 2.0. See [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md) for attribution and sources.
