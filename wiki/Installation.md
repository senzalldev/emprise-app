# Installation

## macOS (Apple Silicon)
Download the signed DMG:
```
curl -fsSL https://raw.githubusercontent.com/senzalldev/emprise-app/main/install.sh | sh
```
Or download directly from [Releases](https://github.com/senzalldev/emprise-app/releases/latest).

## Linux
```
curl -fsSL https://raw.githubusercontent.com/senzalldev/emprise-app/main/install.sh | sh
```

## Windows
Download `emprise-windows-amd64.exe` from [Releases](https://github.com/senzalldev/emprise-app/releases/latest).

## Update
```
emprise update
```
emprise checks for updates on launch and hourly while running.

## Build from Source
```
git clone https://github.com/senzalldev/emprise-app.git
cd emprise-app
go build -o emprise ./cmd/emprise
```

