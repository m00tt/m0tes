# OSQuery
# Intro


# Install options
## Windows
You need to install Chocolately and run the following command:
```powershell
choco install osquery
```

## macOS
osquery can be installed from the Homebrew Cask repository:
```terminal
brew install --cask osquery
```

## Debian
Run the following commands:
```sh
export OSQUERY_KEY=1484120AC4E9F8A1A577AEEE97A80C63C9D8B80B
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys $OSQUERY_KEY
sudo add-apt-repository 'deb [arch=amd64] https://pkg.osquery.io/deb deb main'
sudo apt-get update
sudo apt-get install osquery
```

## RPM Linux
Run the following commands:
```sh
curl -L https://pkg.osquery.io/rpm/GPG | sudo tee /etc/pki/rpm-gpg/RPM-GPG-KEY-osquery
sudo yum-config-manager --add-repo https://pkg.osquery.io/rpm/osquery-s3-rpm.repo
sudo yum-config-manager --enable osquery-s3-rpm-repo
sudo yum install osquery
```

## Other installation options
You can find other installation options here: [OSQuery Download](https://www.osquery.io/downloads)