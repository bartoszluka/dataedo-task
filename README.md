# Przykładowe code-review

Oto fragment kodu, który mam skomentować:

```cs
[HttpPost("delete/{id}")]
public void Delete(uint id)
{
    User user = _context.Users.FirstOrDefault(user => user.Id == id);
    _context.Users.Remove(user);
    _context.SaveChanges();
    Debug.WriteLine($"The user with Login={user.login} has beed deleted.");
    return Ok();
}
```

## Użycie HttpDelete

Pierwszy problem jaki widzę to użycie metody HTTTP POST zamiast DELETE do usunięcia encji z bazy danych.
Poprawka powinna wyglądać tak:

```cs
[HttpDelete("{id}")]
```

## Brak obsługi `null`a

Metoda `FirstOrDefault` zwraca domyślną wartość obiektu w przypadku nie znalezienia pasującego elementu.
Trzeba sprawdzić, czy `user` nie jest `null`em i odpowiednio to obsłużyć.

```cs
User? user = _context.Users.FirstOrDefault(user => user.Id == id);
if (user is null)
{
    return NotFound();
}
```

## Użycie metody `Find` zamiast `FirstOrDefault`

Metoda `Find` jest zoptymalizowana do wyszukiwania obiektów po ich kluczu głównym, natomiast
`FirstOrDefault` jest domyślną metodą na iteratorze.
Z tego powodu w tym przypadku warto użyc metody `Find` zamiast `FirstOrDefault`.

```cs
User? user = _context.Users.Find(id);
```

## Używanie metod asynchronicznych

Podana metoda `Delete` używa synchronicznego API, które blokuje wykonujący wątek.
Zalecane jest używanie API asynchronicznego.
Pozwala to na wznowienie wątku do wykonania innych operacji, w trakcie czekania na odpowiedź z
asynchronicznej metody.

```cs
[HttpPost("delete/{id}")]
public async Task<IActionResult> Delete(uint id)
{
    User user = await _context.Users.FirstOrDefaultAsync(user => user.Id == id);
    _context.Users.Remove(user);
    await _context.SaveChangesAsync();
    Debug.WriteLine($"The user with Login={user.login} has beed deleted.");
    return Ok();
}
```

## Wynik

Ostatecznie według mnie metoda powinna wyglądać tak:

```cs
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(uint id)
{
    User? user = await _context.Users.FindAsync(id);
    if (user is null)
    {
        return NotFound();
    }
    _context.Users.Remove(user);
    await _context.SaveChangesAsync();
    Debug.WriteLine($"The user with Login={user.login} has beed deleted.");
    return Ok();
}
```
