```csharp
public class FluentModelValidator<T> : IModelValidator<T>
{
    private readonly IValidator<T> validator;
    private T subject;

    public FluentModelValidator(IValidator<T> validator)
    {
        this.validator = validator;
    }

    public void Initialize(object subject)
    {
        this.subject = (T)subject;
    }

    public async Task<IEnumerable<string>> ValidatePropertyAsync(string propertyName)
    {
        // If someone's calling us synchronously, and ValidationAsync does not complete synchronously,
        // we'll deadlock unless we continue on another thread.

        //如果同步调用，而 ValidationAsync 不同步完成，
        //将死锁，除非在另一个线程继续。
        return (await this.validator.ValidateAsync(this.subject, propertyName).ConfigureAwait(false))
            .Errors.Select(x => x.ErrorMessage);
    }

    public async Task<Dictionary<string, IEnumerable<string>>> ValidateAllPropertiesAsync()
    {
        // If someone's calling us synchronously, and ValidationAsync does not complete synchronously,
        // we'll deadlock unless we continue on another thread.
        //如果同步调用，而 ValidationAsync 不同步完成，
        //将死锁，除非在另一个线程继续。
        return (await this.validator.ValidateAsync(this.subject).ConfigureAwait(false))
            .Errors.GroupBy(x => x.PropertyName)
            .ToDictionary(x => x.Key, x => x.Select(failure => failure.ErrorMessage));
    }
}
```