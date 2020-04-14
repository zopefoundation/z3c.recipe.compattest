=====================================
Combined runner for multiple packages
=====================================

To run the compatibility tests for the huge amount of individual
packages in isolation we provide a wrapper script which runs all
individual test runners together, but each in a separate process.

It monitors the stdout of those processes and reports back packages with
failures.

    >>> import os, sys

    >>> # Re-use zc.buildout internals in order to handle both windows
    >>> # and other environments.
    >>> from zc.buildout.easy_install import _create_script
    >>> ok_script = os.path.join(sample_buildout, 'test-ok')
    >>> _ = _create_script('''\
    ... #!%s
    ... import time
    ... time.sleep(1)
    ... print('ok') ''' % sys.executable, ok_script)

    >>> failure_script = os.path.join(sample_buildout, 'test-failure')
    >>> _ = _create_script('''\
    ... #!%s
    ... import time
    ... time.sleep(1)
    ... raise SystemError('Fail!') ''' % sys.executable, failure_script)

    >>> from z3c.recipe.compattest.runner import main
    >>> main(1, ok_script, failure_script, no_exit_code=True)
    Running test-ok
    Running test-failure
    test-failure failed with:
    Traceback (most recent call last):
    ...
    SystemError: Fail!
    <BLANKLINE>
    1 failure(s).
    - test-failure

Note that when we pass a number greater than 1 as the first argument,
tests are run in parallel, so the order of output varies.

    >>> main(2, failure_script, ok_script, failure_script, ok_script, \
    ...     no_exit_code=True)
    Running ...
    2 failure(s).
    - test-failure
    - test-failure
