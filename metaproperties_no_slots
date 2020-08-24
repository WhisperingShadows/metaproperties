# file: objects.py
# Copyright 2020 Frank David Martínez Muñoz (mnesarco)
# License: MIT

from typing import Union

__all__ = ('self_properties', 'properties')

def self_properties(self, scope: dict, exclude=(), save_args: bool = False):
    """Copies all items from `scope` to self as attributes with single underscore prefix.

    :param self: instance ref.
    :param scope: dictionary with attributes.
    :param exclude: tuple with names to exclude from `scope`.
    :param save_args: if True, sets self._args with a tuple with `(scope - exclude).values`
    """
    if save_args:
        args = []
        for (k, v) in scope.items():
            if k != 'self' and k not in exclude:
                setattr(self, '_' + k, v)
                args.append(v)
        self._args = tuple(args)
    else:
        for (k, v) in scope.items():
            if k != 'self' and k not in exclude:
                setattr(self, '_' + k, v)

class properties:
    """
    Utilities for building properties with extended features.
    """
    
    def __init__(self, scope: dict, var_name: str, auto_dirty: bool = False):
        
        self._scope = scope
        self._var = var_name
        self._auto_dirty = auto_dirty

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):

        # Clean references
        self._scope = None
        self._var = None

    def prop(self, read_only: bool = False, listener: Union[bool, str] = None, auto_dirty=False):
        """Decorator: Generates a property with additional features.

        :param read_only: if True, only a getter is generated.
        :param listener: if str, changes will fire `self.[listener]`, if bool, changes will fire `self._changed`
        :param auto_dirty: if True, changes will set `self._is_dirty`
        :return: property.
        """

        auto_dirty = self._auto_dirty or auto_dirty
        
        def decorator(f):

            field = '_' + f.__name__

            if read_only and listener:
                raise ValueError(f"property {field} cannot be read_only and observable at the same time.")

            if read_only:
                setter = None
            else:
                if listener:
                    listener_name = listener if isinstance(listener, str) else '_changed'

                    def setter(inst, new):
                        old = getattr(inst, field, None)
                        if old != new:
                            setattr(inst, field, new)
                            if auto_dirty:
                                inst._is_dirty = True
                            (getattr(inst, listener_name))(field, old, new)
                else:
                    def setter(inst, new):
                        if getattr(inst, field, None) != new:
                            setattr(inst, field, new)
                            if auto_dirty:
                                inst._is_dirty = True

            return property(
                lambda inst: getattr(inst, field, f(inst)),
                setter,
                None,
                f.__doc__
            )

        return decorator

# Example usage:
#
# from objects import *
#
# class Car:
#     with properties(locals(), 'meta') as meta:
# 
#         @meta.prop(read_only=True)
#         def brand(self) -> str:
#             """Brand"""
# 
#         @meta.prop(read_only=True)
#         def max_speed(self) -> float:
#             """Maximum car speed"""
# 
#         @meta.prop(listener='_on_acceleration')
#         def speed(self) -> float:
#             "Speed of the car"""
#             return 0  # Default stopped
# 
#         @meta.prop(listener='_on_off_listener')
#         def on(self) -> bool:
#             """Engine state"""
#             return False
# 
#     def __init__(self, brand: str, max_speed: float = 200):
#         self_properties(self, locals())
# 
#     def _on_off_listener(self, prop, old, on):
#         if on:
#             print(f"{self.brand} Turned on, Runnnnnn")
#         else:
#             self._speed = 0
#             print(f"{self.brand} Turned off.")
# 
#     def _on_acceleration(self, prop, old, speed):
#         if self.on:
#             if speed > self.max_speed:
#                 print(f"{self.brand} {speed}km/h Bang! Engine exploded!")
#                 self.on = False
#             else:
#                 print(f"{self.brand} New speed: {speed}km/h")
#         else:
#             print(f"{self.brand} Car is off, no speed change")
# 
# 
# mycar = Car('Ford')
# 
# # Car is turned off
# for speed in range(0, 300, 50):
#     mycar.speed = speed
# 
# # Car is turned on
# mycar.on = True
# for speed in range(0, 350, 50):
#     mycar.speed = speed
