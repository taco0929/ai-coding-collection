# Docstring Style

All public classes, methods, and functions must include docstrings using reStructuredText (Sphinx) style.

Docstrings must include, where applicable:

* concise purpose description
* :param:
* :type:
* :return:
* :rtype:
* :raises:

Requirements:

* Describe intent and behavior, not trivial implementation steps
* Document edge cases and important constraints
* Keep descriptions concise and precise
* Do not generate placeholder or redundant docstrings

Example:

```python
def get_user(user_id: int) -> User:
    """
    Retrieve a user by ID.

    :param user_id: Unique user identifier.
    :type user_id: int
    :return: The matching user entity.
    :rtype: User
    :raises UserNotFoundError: If no user exists for the given ID.
    """
```