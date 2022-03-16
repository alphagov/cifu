# ADR00TBD: Try Rust for an API

Date: 2022-03-15

## Status

`Proposed`

## Context
> The scope of this ADR is to evaluate the technological viability of Rust. Out of scope is if the GDS should adopt it widely, that is a decision for another report, or perhaps an RFC.

Our core API is (or will be) a vital piece of government infrastructure running on GOV.UK PaaS or AWS Lambda. (to be clear, it may or may not be Critical National Infrastructure in the official sense). Though not user-facing, It is important that this API be both reliable and fast, without being complicated to develop. Further, the carbon cost of this infrastructure needs to be considered (in terms of power usage).

### Rust Advantages
Rust bills itself as _"A language empowering everyone to build reliable and efficient software"_. It was first released in 2010, 8 months after Go. In terms of adoption there are about the same number of github projects written in Rust as there are in Go.

Rust is designed to quickly build provably correct software, with an emphasis on zero runtime failures, and high-quality developer ergonomics.

In alpha testing, we wrote a prototype API, powering the X-Gov form builder UI in three languages, the source code for these three prototypes are available in branches of [alphagov/forms-api-prototype](https://github.com/alphagov/forms-api-prototype):
 1. [Ruby](https://github.com/alphagov/forms-api-prototype/pull/1)
 2. [Rust](https://github.com/alphagov/forms-api-prototype/pull/2)
 3. [Python](https://github.com/alphagov/forms-api-prototype/pull/3)

In our prototyping, we saw that Rust is a very good match for the project we planned on building, despite it being a new language to the GDS.

Notable benefits are:
- An [76x decrease](https://github.com/drujensen/fib#results) in execution times (compared to Python, 32x decrease compared to Ruby), and a consummate reduction in power requirement (and therefore carbon cost)
	- Request round-trip times are also faster: The slowest Rust web frameworks [are faster](https://web-frameworks-benchmark.netlify.app/result?asc=0&l=rust,ruby,python]) than the fastest Python and Ruby frameworks. Ruby is 1.45x slower, Python, 1.85x. (only the [sifrr](https://sifrr.github.io/sifrr/#/./packages/server/sifrr-server/) framework is faster in JS land)
- A up to an order of magnitude reduction in memory overhead compared to python and ruby (depending on application) due to lack of runtime overhead.
- Faster development due to instant compiler feedback (instead of waiting for logs)
- Faster development due to less unit testing burden: Many errors that would require unit testing are instead compile-time errors. This is not just simple type errors, but whole categories of invalid states are unrepresentable due to careful language and library design.
- 100% compiled coverage by definition (all compiled code is valid). Allowing the team great confidence that no API responses are invalid, no inputs cause errors, etc.
- A focus in the wider Rust community on correctness and security. Contrast with NPM, which requires so much work to control supply chain attacks that most teams give up the attempt.

### Rust Disadvantages
- Less organisational knowledge. Though PaaS has an [experimental buildpack](https://github.com/alphagov/cf-buildpack-rust) for Rust, and Cyber Security have used it in the past (with one production service operational at time of writing) there are not many projects within GDS.
- Hiring: though we will still hire based on Ruby, Python, and JavaScript, we must advertise that Rust is a language we use, and expect to train new hires.
- Training: Rust has earlier requirements for learning; Whereas most languages wait until scale-up to require tuning, the Rust compiler requires much to be done ahead-of-time. The team can not expect new starters to be expert. 


## Decision

We will build an API in Rust.

## Consequences

To mitigate the disadvantages mentioned above, we will:
- Use the team's 10% L&D time to learn Rust
- Advertise for Rust proficiency in our job listings (in addition to Ruby and JavaScript) 
- Work in the wider GDS community to raise awareness and educate
- Maintain a machine-readable API spec to allow falling-back to a different language should the experiment be a failure