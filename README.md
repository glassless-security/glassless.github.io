# GlaSSLess Website

This is the GitHub Pages website for the [GlaSSLess](https://github.com/glassless-security/glassless) project.

Built using [Roq](https://iamroq.com/), a static site generator powered by Quarkus.

## Development

Run the site locally with live reload:

```bash
./mvnw quarkus:dev
```

The site will be available at http://localhost:8080

## Building

Generate the static site:

```bash
./mvnw quarkus:build -Dquarkus.roq.generator.batch=true
```

The static files will be output to `target/roq/`.

## Deployment

The site is automatically deployed to GitHub Pages when changes are pushed to the `main` branch via GitHub Actions.

## Project Structure

```
.
├── content/           # Site pages (Markdown, HTML)
├── data/              # Site data (YAML files)
├── public/            # Static assets (CSS, images)
├── templates/         # Qute templates
│   ├── layouts/       # Page layouts
│   └── partials/      # Reusable components
└── src/main/resources/
    └── application.properties  # Site configuration
```

## License

Apache License 2.0
