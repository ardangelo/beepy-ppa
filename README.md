# Beepy package repository

## Adding to APT

	curl -s --compressed "https://ardangelo.github.io/beepy-ppa/KEY.gpg" | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/beepy-ppa.gpg >/dev/null
	sudo curl -s --compressed -o /etc/apt/sources.list.d/beepy-ppa.list "https://ardangelo.github.io/beepy-ppa/beepy-ppa.list"
	sudo apt update
