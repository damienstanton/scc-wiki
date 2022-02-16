## List functor as a coroutine suspension functor

While most common functors, some of which ship with [monad-coroutine](), contain only one coroutine resumption, this is no requirement. Here is an example of using the list functor in this role. First, a coroutine:

    factorize :: Monad m => Integer -> Coroutine [] m [Integer]
    factorize n | n < 2 = return []
                | otherwise = suspend (map (\k-> liftM (k:) $ factorize (n `div` k)) (factors n))

    factors n = [k | k <- 2..n | n `mod` k = 0]

So, the coroutine suspends with a list of resumptions. It's the invoker's choice what to do with the list. For example, we can always choose to resume the first alternative only:

    runFirstAlternative :: Monad m => Coroutine [] m r -> m r
    runFirstAlternative cort = pogoStick head cort

    primeFactors :: Integer -> [Integer]
    primeFactors n = runFirstAlternative (factorize n)

As it happens, the first alternative among the `factorize` resumptions always gives us the lowest factors, which must be a prime number. So this resumption strategy results in a list of prime factors.

But why pick only one resumption to resume? We could run all the resumptions and collect the results:

    runAllAlternatives :: Monad m => Coroutine [] m r -> m [r]
    runAllAlternatives cort = resume cort
                              >>= liftM concat . mapM runAllAlternatives

    allFactorizations :: Integer -> [[Integer]]
    allFactorizations = runAllAlternatives (factorize n)

The result of the function `allFactorizations` is a list of lists of factors of its argument, where the product of every list of factors equals the argument.
