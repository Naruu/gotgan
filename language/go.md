### byte, string, rune

- string is a read-only slice of byte array
- rune is a code code point, which is an alias for the int32, asw well as character constant
- declare byte, rune with single quote, string with double quote

https://go.dev/blog/strings

- Go source code is always UTF-8.
- A string holds arbitrary bytes.
- A string literal, absent byte-level escapes, always holds valid UTF-8 sequences.
- Those sequences represent Unicode code points, called runes.
- No guarantee is made in Go that characters in strings are normalized.
- Iterating with range loops gives rune, whereas indexing with index gives byte
  ```
  // rune type
  const nihongo = "日本語"
  for index, runeValue := range nihongo {
      fmt.Printf("%#U starts at byte position %d\n", runeValue, index)
  }
  ```
  ```
  // byte type
      for i := 0; i < len(sample); i++ {
      fmt.Printf("%x ", sample[i])
  }
  ```
