# Shamir's Secret Sharing Scheme Implementation

# Code Implementation

```python
import random
from functools import reduce

PRIME = 2 ** 127 - 1

def eval_polynomial(coeffs, x, prime=PRIME):

    """Evaluating a polynomial with its coefficient at x modulo prime."""
    result = 0
    for power, coeff in enumerate(coeffs):
        result = (result + coeff * pow(x, power, prime)) % prime
    return result


def generate_coefficients(secret, k, prime=PRIME):
    """Generates random  coefficient with constant term as secret."""
    coeffs = [secret] + [random.randrange(0, prime) for _ in range(k - 1)]
    return coeffs


def split_secret(secret, k, n, prime=PRIME):

    """Splitting the secret into n shares with k."""
    coeffs = generate_coefficients(secret, k, prime)
    shares = []
    for i in range(1, n + 1):
        x = i
        y = eval_polynomial(coeffs, x, prime)
        shares.append((x, y))
    return shares


def reconstruct_secret(shares, prime=PRIME):

    """Reconstructing the secret from shares byLagrange Interpolation."""
    def _lagrange_basis(j, x_values):
        numerator, denominator = 1, 1
        xj = x_values[j]
        for m, xm in enumerate(x_values):
            if m != j:
                numerator = (numerator * -xm) % prime
                denominator = (denominator * (xj - xm)) % prime
        return numerator * pow(denominator, -1, prime)

    x_values = [x for x, _ in shares]
    y_values = [y for _, y in shares]
    secret = 0
    for j in range(len(shares)):
        lj = _lagrange_basis(j, x_values)
        secret = (prime + secret + (y_values[j] * lj)) % prime
    return secret


if __name__ == "__main__":
    secret = 9876543210123456789
    k = 3  
    n = 6  # Total shares

    print(f"Secret to share: {secret}")

    shares = split_secret(secret, k, n)
    print("\nGenerated shares:")
    for s in shares:
        print(s)

    # Picking any k shares to reconstruct
    sample = random.sample(shares, k)
    reconstructed = reconstruct_secret(sample)

    print(f"\nReconstructed Secret: {reconstructed}")
    assert reconstructed == secret
