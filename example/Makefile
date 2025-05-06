default: build
.PHONY: build clean serve

build:
	git clone https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke || true
	git clone git@github.com:nbr23/hugo-shortcodes-tts.git themes/hugo-shortcodes-tts || true
	docker compose up tts-api --wait
	docker compose run --rm hugo
	docker compose down

serve:
	git clone https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke || true
	git clone git@github.com:nbr23/hugo-shortcodes-tts.git themes/hugo-shortcodes-tts || true
	docker compose up tts-api --wait
	docker compose up tts-api hugo-serve

clean:
	docker compose down
	rm -rf .voices public themes
