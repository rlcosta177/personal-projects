install aws via curl

change the aws config and credentials:
 1) ./aws -> nano config:
	```bash
	[default]
	region = us-east-1
	output = json
	```

[profile vocareum] -> este profile é criado 
	```bash
	region = us-east-1
	output = json
	source_profile = default
	```
	
 2) ./aws -> nano credentials(os valores têm que ser alterados sempre que iniciar a aws account(os tres valores são temporários)):
	```bash
	[default]
	aws_access_key_id=ASIAU6GDWVFNZNBAHGYL
	aws_secret_access_key=HRWy4ojia/hYUH/A5vNtmUjFXtzOQ8dmlUN4C+Lg
	aws_session_token=FwoGZXIvYXdzEJP//////////wEaDKrgyaHVBywkqwIgSSK+Af52dQmPAmrnD7dUG/zPBYMM4PCpbl7c20jxa/Qm2SHLaLJfz2SumDFD/vqYLr8Uf>
	```

	```bash
	[vocareum]
	aws_access_key_id=ASIAU6GDWVFNZNBAHGYL
	aws_secret_access_key=HRWy4ojia/hYUH/A5vNtmUjFXtzOQ8dmlUN4C+Lg
	aws_session_token=FwoGZXIvYXdzEJP//////////wEaDKrgyaHVBywkqwIgSSK+Af52dQmPAmrnD7dUG/zPBYMM4PCpbl7c20jxa/Qm2SHLaLJfz2SumDFD/vqYLr8Uf>
	```


- install terraform and do -> cp terraform /usr/local/bin
- Dentro da pasta do security_onion_jackdaniels/cybersecurity no terminal(pode ser o intelij ou terminal normal):
    - `terraform init`
    - `terraform plan` -> pass que eu quizer para as maquinas
    - `terraform apply` -> inserir as passwords criadas no terraform plan (o professor percisa de ter partilhado as AMIs primeiro)
