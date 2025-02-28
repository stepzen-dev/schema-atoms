ATOMS=../../atoms

schema:
	mkdir -p atoms
	cp -r $(ATOMS)/delivery atoms

deploy: schema
	stepzen deploy

clean:
	rm -fr atoms
