# SK.Xbox360.CPUKey

[![.NET 9.0](https://img.shields.io/badge/.NET-9.0-purple.svg)](https://dotnet.microsoft.com/download/dotnet/9.0)
[![NuGet](https://img.shields.io/nuget/v/SK.Xbox360.CPUKey.svg)](https://www.nuget.org/packages/SK.Xbox360.CPUKey/)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

An immutable .NET data type for Xbox 360 CPUKeys, offering value semantics with optimized parsing, validation, conversion, and utility operations. Designed for high-performance scenarios with zero-allocation span-based APIs and suitable for use in collections requiring fast look-up, equality checks, and sorting.

The `CPUKey` public interface follows the same familiar dotnet API usage patterns as built-in types like `Guid`, with both strict parsing (throwing exceptions on failure) and safe parsing (returning `bool` and `out` parameter) methods.

A `CPUKey` object is stored as an immutable fixed-size 16-byte array, with no heap allocations after construction, and providing the same value-type semantics as built-in types. `ToString()` implements a lazy string conversion using an optimized custom implementation of `Convert.ToHexString` that avoids redundant exception handling, and minimizes arithmetic and branching using a static lookup table.

All equality, comparison, and hashing operations are performed directly on the underlying byte array or in `stackalloc` buffers for optimal performance.

Sanity checks and integrity validation are performed during initialization to ensure that only cryptographically valid CPUKeys can be constructed, with specialized exception types for different failure modes. Validation includes length checks, non-zero payload, Hamming weight, and ECD (Error Correction and Detection) bits.

## Quick Start

### Basic usage

```cs
using SK.Xbox360;

// Initialize from hex strings (case-insensitive), byte arrays, spans, or char spans
byte[] arr = [0xC0, 0xDE, 0x8D, 0xAA, 0xE0, 0x54, 0x93, 0xBC, 0xB0, 0xF1, 0x66, 0x4F, 0xB1, 0x75, 0x1F, 0x00];
string str = Convert.ToHexString(arr); // "C0DE8DAAE05493BCB0F1664FB1751F00"

// Throwing parse (throws exceptions on failure)
var parsedKey = CPUKey.Parse(arr);

// Non-throwing parse (returns bool, outputs CPUKey or null/Empty)
if (CPUKey.TryParse(str, out var cpukey))
	Console.WriteLine($"Valid CPUKey: {cpukey}");

// Trivially perform equality/comparison between keys, byte arrays, hex strings, or char spans
if (cpukey == new CPUKey("c0de8daae05493bcb0f1664fb1751f00"))
	Console.WriteLine($"CPUKeys match ({parsedKey})."); // Outputs: CPUKeys match (C0DE8DAAE05493BCB0F1664FB1751F00).

// Generate a cryptographically-random valid CPUKey
var randomKey = CPUKey.CreateRandom();

// Use in collections
var keySet = new HashSet<CPUKey> { parsedKey, randomKey };
var keyDict = new Dictionary<CPUKey, string> {
	[parsedKey] = "Console #1",
	[randomKey] = "Console #2"
};
```

### Advanced usage: LINQ queries, database ORMs, network streams

```cs
public static class DBExtensions
{
	// ORM helper with LINQ-to-SQL (PetaPoco, Dapper, Entity Framework, etc.)
	public static Client? GetClient(this IDatabaseService dbService, CPUKey cpukey)
		=> dbService.DB.SingleOrDefault<Client>(x => x.CPUKey == cpukey);
}

public class Client
{
	public int Id { get; set; }
	public CPUKey CPUKey { get; set; }
	//... other properties like Gamertag, IP, etc.
}

internal class PacketHandler
{
	public bool HandleClientData(EndianReader reader, IDatabaseService db)
	{
		if (!CPUKey.TryParse(reader.ReadBytes(16), out var cpukey) || cpukey is null)
		{
			Console.WriteLine("Connection contains invalid CPUKey data");
			return false;
		}

		if (db.GetClient(cpukey) is not Client client)
		{
			Console.WriteLine($"Connection from unknown client with CPUKey {cpukey}");
			return false;
		}

		// Handle client connection...
		Console.WriteLine($"Connection from client {client.Id} with CPUKey {client.CPUKey}");
		return true;
	}
}
```

### Parsing and Validation

```csharp
// Safe parsing - distinguishes malformed vs invalid
string[] testInputs = {
	"C0DE8DAAE05493BCB0F1664FB1751F00", // Valid
	"STELIOKONTOSCANTC0DECLIFTONMSAID", // Malformed (non-hex characters)
	"C0DE8DAAE05493BCB0F1664FB1751F01"  // Invalid (wrong ECD/Hamming weight)
};

foreach (string input in testInputs)
{
	if (CPUKey.TryParse(input, out CPUKey? cpukey))
		Console.WriteLine($"✓ Valid: {cpukey}");
	else if (cpukey is null)
		Console.WriteLine($"✗ Malformed input: {cpukey}");
	else if (!cpukey.IsValid())
		Console.WriteLine($"✗ Well-formed but invalid: {cpukey}");
}
```

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE.md](LICENSE.md) file for details.

---

**Made with ❤️ by Stelio Kontos for the Xbox 360 homebrew community**

For questions, issues, or contributions, please visit the [GitHub repository](https://github.com/Ste1io/SK.Xbox360.CPUKey).
