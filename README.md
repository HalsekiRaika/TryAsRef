## AsRef/AsMut version of TryFrom

### Usage
```rust
    struct TestStruct(TestEnum);

    #[derive(Debug, thiserror::Error)]
    enum TestError {
        #[error(transparent)] 
        InvalidType(anyhow::Error) 
    }

    impl TryAsRef<str> for TestStruct {
        type Error = TestError;
        fn try_as_ref(&self) -> Result<&str, Self::Error> {
            match &self.0 {
                TestEnum::AType(s) => Ok(s),
                TestEnum::BType(_) => Err(TestError::InvalidType(anyhow::Error::msg("cannot get reference to str"))),
            }
        }
    }

    impl TryAsRef<BType> for TestStruct {
        type Error = TestError;
        fn try_as_ref(&self) -> Result<&BType, Self::Error> {
            match &self.0 {
                TestEnum::AType(_) => Err(TestError::InvalidType(anyhow::Error::msg("cannot get reference to BType"))),
                TestEnum::BType(b) => Ok(b),
            }
        }
    }

    impl TryAsMut<str> for TestStruct {
        type Error = TestError;
        fn try_as_mut(&mut self) -> Result<&mut str, Self::Error> {
            let s = match &mut self.0 {
                TestEnum::AType(ref mut s) => s,
                TestEnum::BType(_) => return Err(TestError::InvalidType(anyhow::Error::msg("cannot get mutable reference to str"))),
            };
            Ok(s)
        }
    }

    impl TryAsMut<BType> for TestStruct {
        type Error = TestError;
        fn try_as_mut(&mut self) -> Result<&mut BType, Self::Error> {
            let s = match &mut self.0 {
                TestEnum::AType(_) => return Err(TestError::InvalidType(anyhow::Error::msg("cannot get mutable reference to BType"))),
                TestEnum::BType(ref mut b) => b,
            };
            Ok(s)
        }
    }

    enum TestEnum { AType(String), BType(BType) }
    struct BType();

    fn test_try_as_ref() -> anyhow::Result<()> {
        let a = TestStruct(TestEnum::AType(String::from("a_type")));
        let a: Result<&str, TestError> = a.try_as_ref();
        assert!(a.is_ok());

        let a: TestStruct = TestStruct(TestEnum::BType(BType()));
        let a: Result<&str, TestError> = a.try_as_ref();
        assert!(a.is_err());

        let b = TestStruct(TestEnum::BType(BType()));
        let b: Result<&BType, TestError> = b.try_as_ref();
        assert!(b.is_ok());

        let b = TestStruct(TestEnum::AType(String::from("a_type")));
        let b: Result<&BType, TestError> = b.try_as_ref();
        assert!(b.is_err());
        Ok(())
    }

    fn test_try_as_mut() -> anyhow::Result<()> {
        let mut a = TestStruct(TestEnum::AType(String::from("a_type")));
        let a: Result<&mut str, TestError> = a.try_as_mut();
        assert!(a.is_ok());

        let mut a: TestStruct = TestStruct(TestEnum::BType(BType()));
        let a: Result<&mut str, TestError> = a.try_as_mut();
        assert!(a.is_err());

        let mut b = TestStruct(TestEnum::BType(BType()));
        let b: Result<&mut BType, TestError> = b.try_as_mut();
        assert!(b.is_ok());

        let mut b = TestStruct(TestEnum::AType(String::from("a_type")));
        let b: Result<&mut BType, TestError> = b.try_as_mut();
        assert!(b.is_err());
        Ok(())
    }
```