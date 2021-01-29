## Pancake Example

#### PORT
```
Pancake = {
	fromMixture: () => Pancake = { return Pancake(); }
};

PancakeMixture = {
	mixFromMaterials: () => PancakeMixture = { return PancakeMixture(); }
};

PancakePlate = {
	pancakes: Pancake*;
};


PancakeMaker = !{
	giveMePancakes: (amount: Int) => PancakePlate;	
};

PancakeMixDispenser = !{
	giveMeSomeMixture: () => PancakeMixture;
};

PancakePan = !{
	bake: (mixture: PancakeMixture, amount: Int) => PancakePlate;
};
```

#### USE CASE

```
MasterPancakeMaker = {
	implements PancakeMaker;
	
	mixDispenser: PancakeMixDispenser!;
	pan: PancakePan!;
	
	giveMePancakes = {
		return pan.bake(mixDispenser.giveMeSomeMixture(), amount);
	}

};
```

#### ADAPTERS
```
TeflonPan = {
	implements PancakePan;
	
	bake = {
		return PancakePlate(
			pancakes: 1.rangeTo(amount).map(() => {
				return Pancake.fromMixture(mixture);
			})
		)
	}
};

CoolMixDispenser = {
	implements PancakeMixDispenser;
	
	giveMeSomeMixture = {
		return PancakeMixture.mixFromMaterials();
	};
};

WebPancakeService = {
	pancakeMaker: PancakeMaker!;
	
	handle: (getData: Any#) => WebResponse = {
		amount: Amount = Int(getData['amount'], 1);
		plate: Plate = pancakeMaker.giveMePancakes(amount);
		return WebResponse(200, "Your plate is ready...");
	}
};

CliPancakeService = {
	pancakeMaker: PancakeMaker!;
	
	handle: (argv: String*) => CliResponse = {
		amount: Amount = Int(argv[1], 1);
		plate: Plate = pancakeMaker.giveMePancakes(amount);
		return CliResponse("Your plate is ready...");
	}
};
```
