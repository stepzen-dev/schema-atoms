ATOMS=../../atoms

schema:
	mkdir -p atoms/fast-package
	cp -r $(ATOMS)/fast-package/core atoms/fast-package

deploy: schema
	stepzen deploy

clean:
	rm -fr atoms
